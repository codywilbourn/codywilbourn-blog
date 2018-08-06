---
author: Cody
comments: false
type: page
title: Contact
---

Three options on how to contact me

  1. Submit the contact form here
  2. Send me an email -- first name at this domain.
  3. Message me on [Twitter](http://twitter.com/codywilbourn)

<form class="black-80" name="contact" method="POST" netlify-honeypot="missing" netlify>
  <label for="name" class="f6 b db mb2">Name <span class="normal black-60">(required)</span></label>
  <input id="name" name="name" class="input-reset ba b--black-20 pa2 mb2 db w-100" type="text" required />

  <label for="email" class="f6 b db mb2">Email <span class="normal black-60">(required)</span></label>
  <input id="email" name="email" class="input-reset ba b--black-20 pa2 mb2 db w-100" type="email" required />

  <label for="message" class="f6 b db mb2">Message <span class="normal black-60">(required)</span></label>
  <textarea id="message" name="message" class="input-reset ba b--black-20 pa2 mb2 db w-100" required> </textarea>

  <p class="dn">
    <label>Leave this field blank <input name="missing" /></label>
  </p>
  <p>
    <button type="submit" class="button-reset">Send</button>
  </p>
</form>
