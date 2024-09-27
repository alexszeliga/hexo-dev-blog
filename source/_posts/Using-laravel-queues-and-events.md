---
title: Using Laravel Queues, Jobs, and Broadcasting Events.
date: 2024-08-28 14:59:49
tags: [laravel,php,development]
---
### What is a queue and why are they useful

PHP is single threaded, so concurrency is managed in the software layer. Using Laravel's queue, developers have a first class abstraction for asynchronus behavior. The queue helps the running application to prioritize and execute code asynchronously. One of the nice things about Laravel's implementation of the queue is the ability to refactor a performance bottle neck into a queued job.

Imagine your working on a social media application which has `Communities`, and `Users`. `Users` can add `Conversations` to a `Community`. `Users` can then add a `Comment` to a `Conversation` or another `Comment` by filling out a small form built with Livewire and Volt.

```PHP
...
new class extends Component {
    #[Validate('required')]
    public string $body;

    public Conversation $conversation;
    public $root;

    public function submit() 
    {
        $this->validate();
        Comment::create([
            'conversation_id' => $this->conversation->id,
            'user_id' => Auth::user()->id,
            'commentable_id' => $this->root->id,
            'commentable_type' => $this->root::class,
            'body' => $this->body,
        ]);
        $this->dispatch('comment-created', rootId: $this->root->id);
    }
}; ?>

<form wire:submit="submit" class="space-y-6">
    <div>
        <x-input-label>Add Comment</x-input-label>
        <x-text-input class="block w-full" wire:model="body" />
        <x-input-error :messages="$errors->get('body')"/>
    </div>
    <x-primary-button>Submit</x-primary-button>
</form>
```
Let's say this results in a performance bottleneck in our application. We can extract the behavior from the front end and use a queued job triggered from the frontend, and allow the application to insert the new comment asynchronously.

Since a comment can be left on a conversation or a comment, I'm using Laravel's polymorphic relationships to pass the `$root` of the comment into the constructor.

Jobs can be created using an artisan command, which I recommend: `php artisan make:job`. You can pass a name in, or artisan will ask you for one. In this case, I named it `InsertComment`, and extracted the behavior from the ui  component.
```PHP
...
class InsertComment implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Conversation $conversation,
        public User $user,
        public Model $root,
        public string $body,
    )
    {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Comment::create([
            'conversation_id' => $this->conversation->id,
            'user_id' => $this->user->id,
            'commentable_id' => $this->root->id,
            'commentable_type' => $this->root::class,
            'body' => $this->body,
        ]);
    }
}
```
...and then we can update our form component to use the new Job by firing the `::dispatch()` static method, passing in the parameters for the constructor. The comment root is enumerated into it's id and class, so it doesn't need to be typed specifically, but this gives future developers some guide as to what a "root" may be, in context. 
```PHP
...
new class extends Component {
    #[Validate('required')]
    public string $body;

    public Conversation $conversation;
    public $root;

    public function submit() 
    {
        $this->validate();
        InsertComment::dispatch(
            $this->conversation,
            Auth::user(),
            $this->root,
            $this->body,
        );
        // $this->dispatch('comment-created', rootId: $this->root->id);
    }
}; ?>

<form wire:submit="submit" class="space-y-6">
    <div>
        <x-input-label>Add Comment</x-input-label>
        <x-text-input class="block w-full" wire:model="body" />
        <x-input-error :messages="$errors->get('body')"/>
    </div>
    <x-primary-button>Submit</x-primary-button>
</form>
```
As you can see, I've commented out the code that would previously dispatch a browser event after a comment was created. In the old code, that was fine. Single-threaded-PHP would interpret the code line-by-line and by the time the browser event was firing, we could be sure a new comment existed in the DB, but now we cannot be so sure. All things considered, the application may prioritize the generation of the browser event over the insertion of the comment. So the next step is to broadcast the event to our clients. While it might seem like the job is a perfectly good place to broadcast from, one of the other superpowers of jobs on the queue is they are isolated to maximize performance. The end result is that any data that you didn't inject into the job's constructor will not be available. The queue has access to the application in memory, but likely won't be able to track any changes in the app state.

### Broadcasting server events to a Livewire component

Laravel's broadcasting capabilities, as of Laravel 11, are no longer configured by default. Websockets and RTC are managed by a separate server alongside your web server, and that server can either be managed by you or you can use a 3rd party provider to serve your websocket connections. If you already have websockets and broadcasting configured on your app, you can move on, but if not you may want to review [Laravel's docs](https://laravel.com/docs/11.x/broadcasting) on broadcasting. If you are starting from scratch, I would recommend using [Laravel Reverb](https://laravel.com/docs/11.x/reverb).

Broadcasting is related to the queue, as the queue is tasked with prioritizing the events you are broadcasting, but a broadcast event can be received by a user client, and then the user can request a fresh set of data and a new UI, which is how they differ from a job on the queue. Similar to a job, an event can be set up using an artisan command: `php artisan make:event`.

After creating the event, I set it up as follows:

```PHP
...
class CommentCreated implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public string $rootId,
    )
    {}

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\Channel
     */
    public function broadcastOn(): Channel
    {
        return 
            new Channel('comment.' . $this->rootId);
    }
}
```

I made two changes to the class as created by artisan.

1. I had the class implement `ShouldBroadcast`
2. I changed the `broadcastOn` method to return a single channel instead of an array of channels, with the default being private.

For a chat or other peer-to-peer style socket communication, you will want to make sure that the correct client is being communicated with and that they are authorized to be on the channel, but for updating a web page after an insert, using a public channel allows comments created by one user to show up in real-ish time for a second user viewing the same page. 

I then modified the handle method on the job to queue the event after the comment is inserted:
```php
...
use App\Events\CommentCreated;
...
class InsertComment implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Conversation $conversation,
        public User $user,
        public Model $root,
        public string $body,
    )
    {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Comment::create([
            'conversation_id' => $this->conversation->id,
            'user_id' => $this->user->id,
            'commentable_id' => $this->root->id,
            'commentable_type' => $this->root::class,
            'body' => $this->body,
        ]);
        CommentCreated::dispatch($this->root->id);
    }
}
```
Your app needs routes set up for broadcasting, so when you configure your app for broadcasting, a `channels.php` file is added to your routes directory.
```PHP
use Illuminate\Support\Facades\Broadcast;

Broadcast::channel('App.Models.User.{id}', function ($user, $id) {
    return (int) $user->id === (int) $id;
});

Broadcast::channel('comment.{rootId}', function ($rootId) {
    return true;
});
```
The route I added gets a route name as the first parameter. This will be used later to identify which element may need to be updated. The method passed in as the second parameter should return true or false to determine if the client is authorized. Similar to a job, any data required to do this would have to be passed in, because extracting data from the application in memory is not trustworthy "on the queue". Now we can go back and modify our Livewire components to detect the event and update the component. Here's the view that was previously detecting the browser event to display a new comment. The form we looked at previously is pulled in on line 49. The method to handle the event is on line 7.
```php
...
new #[Layout('layouts.app')] class extends Component {
    public Conversation $conversation;
    public bool $showRootComment = false;

    #[On('comment-created')]
    public function commentCreated($rootId) {
        if ($this->conversation->id === $rootId) {
            $this->reset(['showRootComment']);
            $this->conversation->refresh();
        }
    }

    public function mount(Community $community, Contribution $contribution) {
        $this->conversation = Conversation::where('community_id', '=', $community->id)
                                          ->where('contribution_id', '=', $contribution->id)
                                          ->sole();
    }

    public function with() {
        return [
            'comments' => $this->conversation->comments()->paginate(10),
        ];
    }

}; ?>

<div>
    <x-header>
        <div class="flex items-center justify-between">
            <x-h1>
                {{ $conversation->contribution->name }}
            </x-h1>
            <div class="flex flex-col space-y-1">
                @if($conversation->community->userIsSubscribed(Auth::user()))
                <x-primary-button wire:click="$toggle('showRootComment')">
                    Comment
                </x-primary-button>
                <livewire:components.turn.toggle :root="$conversation" key="vote-{{$conversation->id}}" />
                @endif
                <x-secondary-button-link :href="$conversation->contribution->url" target="_BLANK">
                    Visit
                </x-secondary-button-link>
            </div>
        </div>
    </x-header>
    @if($showRootComment)
    <x-content-card>
        <livewire:components.comment.form :conversation="$conversation" :root="$conversation" :key="$conversation->id" />
    </x-content-card>
    @endif
    @if($comments->count())
    <x-content-card>
        <x-h2>The Conversation</x-h2>
        <div class="space-y-6">
            @foreach($comments as $comment)
                <livewire:components.comment.card 
                    :comment="$comment"
                    :conversation="$conversation"
                    :root="$comment"
                    :key="$comment->id"/>
            
            @endforeach
        </div>
        {{ $comments->links() }}
    </x-content-card>
    @endif
</div>
```

Under the hood of Reverb, Laravel is using a package called `echo` to broadcast to the client browsers. Livewire can detect these events and react to them just like any other browser event, so you can modify the above code as follows to have it track the browser event that is generated when the server event is broadcast:
{% codeblock lang:php first_line:2 %}
...
new #[Layout('layouts.app')] class extends Component {
    public Conversation $conversation;
    public bool $showRootComment = false;

    public function getListeners() {
        return [
            "echo:comment.{$this->conversation->id},CommentCreated" => 'commentCreated'
        ];
    }

    public function commentCreated() {
        $this->reset(['showRootComment']);
        $this->conversation->refresh();
    }
...
{% endcodeblock %}
So this closes the loop: we queued a job, triggered by a user, when the job finished, it queued an event. The event was broadcast active sessions, and the component refreshes it's state, although to call it completely finished you will probably want to show some kind of loading state in your UI when kicking off the job that can be cleared when the event fires and is broadcast.
<img src="https://i.imgflip.com/91uku8.jpg"/>