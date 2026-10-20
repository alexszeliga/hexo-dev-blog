---
title: The Cheese Party
date: 2026-09-29 12:00:00
---
It's that time of year again. I will note that the days seem to be getting shorter. I bet if we kept track over a couple years, we could probably figure out when the shortest one will be every year. Huh.

Anyway: Cheese. It's the time of year where we start talking about cheese. Lois works on cheese puns. Alex actively avoids Dominic The Donkey. I can't think of anything else that tends to happen around this time of year so since you're probably not busy, we should have a party about it.

# Save the Date: Saturday, December 5th, 2026

- **When:** Saturday, December 5th, 2026
  - It's an open-house. Come any time after 2pm for family friendly cheese and fun. Stay after 6pm for more traditional cheese party excitement!
- **Where:** Alex and Lois' house, Philadelphia

That's all the details you need for now. If you have any questions, reach out to Alex or Lois directly.

## What's The Cheese Party

The cheese party is a tradition that goes back over 10 years. Alex and Lois buy way too much wine and open their house for a day of community, joy, and cheese.

The only rule to cheese party is that you must bring cheese. You can bring a dish that includes cheese, or you can bring a cheese board, or you can bring a cheese with a side or cracker. You may bring wine, but you do not have to. You might want to bring beer if that's what you're into.

Cheese Party is not a competition. Cheese Party brings people together, and competition is just a wedge between friends, so be assured that Cheese Party is not a competition. But if it were a competition, Alex will probably declare a winner before the end of the evening.

## Will You Be There?

Let us know if you're coming so we can plan accordingly!

<style>
.cheese-party-form {
  max-width: 480px;
}
.cheese-party-form .form-field {
  margin-bottom: 1rem;
}
.cheese-party-form label {
  display: block;
  font-weight: bold;
  margin-bottom: 0.25rem;
}
.cheese-party-form input[type="text"],
.cheese-party-form input[type="email"],
.cheese-party-form input[type="tel"],
.cheese-party-form select {
  width: 100%;
  box-sizing: border-box;
  padding: 0.5rem;
  font-size: 1rem;
}
.cheese-party-form button {
  padding: 0.6rem 1.5rem;
  font-size: 1rem;
  cursor: pointer;
}
.cheese-party-form .form-message {
  margin-top: 1rem;
  font-weight: bold;
}
.cheese-party-form .form-hint {
  margin-top: -0.5rem;
  margin-bottom: 1rem;
  font-size: 0.9rem;
  color: #555;
}
.cheese-party-form .form-message.error {
  color: #cc0000;
}
.cheese-party-form .form-message.success {
  color: #009900;
}
</style>

<form class="cheese-party-form" id="cheese-party-form">
  <div class="form-field">
    <label for="cheese-party-name">Name</label>
    <input type="text" id="cheese-party-name" name="name" required>
  </div>
  <div class="form-field">
    <label for="cheese-party-email">Email</label>
    <input type="email" id="cheese-party-email" name="email">
  </div>
  <div class="form-field">
    <label for="cheese-party-phone">Phone</label>
    <input type="tel" id="cheese-party-phone" name="phone">
  </div>
  <p class="form-hint">Provide at least one way to contact you: email or phone.</p>
  <div class="form-field">
    <label for="cheese-party-party_size">Party Size</label>
    <select id="cheese-party-party-size" name="party_size" required>
      <option value="1">1</option>
      <option value="2">2</option>
      <option value="3">3</option>
      <option value="4">4</option>
      <option value="5">5</option>
    </select>
  </div>
  <div class="form-field">
    <label for="cheese-party-contact">OK to Contact You About the Party?</label>
    <select id="cheese-party-contact" name="contact" required>
      <option value="Yes">Yes</option>
      <option value="No">No</option>
      <option value="1 Week Warning">1 Week Warning</option>
    </select>
  </div>
  <button type="submit">RSVP</button>
  <p class="form-message" id="cheese-party-form-message" role="status"></p>
</form>

<script>
(function () {
  var form = document.getElementById('cheese-party-form');
  var message = document.getElementById('cheese-party-form-message');
  if (!form) return;

  form.addEventListener('submit', function (event) {
    event.preventDefault();
    message.className = 'form-message';

    // Email or phone is required (but not both): validate before submitting.
    var email = form.elements['email'].value.trim();
    var phone = form.elements['phone'].value.trim();
    if (!email && !phone) {
      message.className = 'form-message error';
      message.textContent = 'Please provide an email or a phone number so we can reach you.';
      return;
    }

    message.textContent = 'Sending...';

    // The Apps Script endpoint parses a JSON body (JSON.parse(e.postData.contents)).
    // Note: Apps Script cannot send CORS headers, so the response is "opaque"
    // and we cannot read the {"result": ...} payload. We rely on the request
    // being delivered (network success) and treat it as success.
    var fields = new FormData(form);
    var payload = {};
    fields.forEach(function (value, key) { payload[key] = value; });

    // Content-Type must be 'text/plain' (a "simple" header) or the browser
    // would have to preflight, and Apps Script cannot answer preflights.
    // Setting a simple header keeps it a single POST; the JSON body itself
    // is unaffected — the script reads it from e.postData.contents.
    fetch('https://script.google.com/macros/s/AKfycbyTBwS-Nu4JfNEwcQc4UpfbARYbIaL1opzdsf-WNlhFp6pmScH9IsdALvotQYamXGcH/exec', {
      method: 'POST',
      mode: 'no-cors',
      headers: { 'Content-Type': 'text/plain;charset=utf-8' },
      body: JSON.stringify(payload)
    }).then(function () {
      message.className = 'form-message success';
      message.textContent = 'Thanks! Your RSVP is in.';
      form.reset();
    }).catch(function () {
      message.className = 'form-message error';
      message.textContent = 'Sorry, something went wrong. Please try again, or email us directly.';
    });
  });
})();
</script>
