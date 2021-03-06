---
title: Contacts
old_permalink: /versions/v10.0.0/sdk/contacts.html
previous___FILE: ./constants.md
next___FILE: ./facebook.md
---

Provides access to the phone's system contacts.

### `Exponent.Contacts.getContactsAsync(fields)`

Get a list of all entries in the system contacts. This returns the name and optionally phone number and email of each contact.

#### Arguments

-   **fields (_array_)** -- An array describing fields to retrieve per contact. Each element bust be one of `Exponent.Contacts.PHONE_NUMBER` or `Exponent.Contacts.EMAIL`.

#### Returns

An array of objects of the form `{ id, name, phoneNumber, email }` with `phoneNumber` and `email` only present if they were requested through the `fields` parameter.

#### Example

```javascript
async function showFirstContactAsync() {
  const contacts = await Exponent.Contacts.getContactsAsync([
    Exponent.Contacts.PHONE_NUMBER,
    Exponent.Contacts.EMAIL,
  ]);
  if (contacts.length > 0) {
    Alert.alert(
      'Your first contact is...',
      `Name: ${contacts[0].name}\n` +
      `Phone: ${contacts[0].phoneNumber}\n` +
      `Email: ${contacts[0].email}`
    );
  }
}
```

This function will display the first entry in the user's contacts.
