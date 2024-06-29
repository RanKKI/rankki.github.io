---
title: "Use Firebase remote config with Expo"
date: 2024-06-20T13:15:18+11:00
draft: false
tags:
  - Expo
  - Firebase
categories:
lang: en
---

[Firebase JS SDK](https://docs.expo.dev/guides/using-firebase/#using-firebase-js-sdk) does not support Remote Config by default[^1]. The only way to use it is to eject from Expo and use a bare project with [React Native Firebase](https://rnfirebase.io/).

Refactoring the current project to use a different approach is challenging, so we prefer to stay with Expo. 

---

To use Firebase Remote Config, we can follow the [instructions](https://firebase.google.com/docs/remote-config/get-started?platform=web), but this leads to the first problem:

```
firebaseerror: remote config: indexed db is not supported by current browser (remoteconfig/indexed-db-unavailable).
```

## IndexedDB

IndexedDB is a low-level API for client-side storage of significant amounts of structured data, including files/blobs[^2]. Currently, React Native (RN) does not support IndexedDB[^3].

### Mock

We can use `fakeIndexedDB`[^4] to override `window.indexedDB` to simulate its presence.

```javascript
import "fake-indexeddb/auto";
```

This leads to another error:

```
structuredClone is not defined
```

### structuredClone

structuredClone is one of the [`Non-ECMAScript JS APIs`](https://github.com/facebook/hermes/discussions/1072#discussioncomment-6578204)[^5], but it is required for `fakeIndexedDB`, We can mock this in the environment as well.

```javascript
import { cloneDeep } from "lodash";
window.structuredClone = cloneDeep;
```

## Firebase Error

We can now continue with Firebase, but another error occurs when calling the `fetchAndActivate` function:

```
failed to fetch and activate remote config FirebaseError: Installations: Could not process request. Application offline.
```

This happens because Firebase checks navigator.onLine to ensure the app is online:

```javascript
if (installationEntry.registrationStatus === RequestStatus.NOT_STARTED) {
  if (!navigator.onLine) {
    // Registration required but app is offline.
    const registrationPromiseWithError = Promise.reject(
      ERROR_FACTORY.create(ErrorCode.APP_OFFLINE)
    );
    return {
      installationEntry,
      registrationPromise: registrationPromiseWithError
  };
}
```

navigator.onLine returns the online status of the browser[^6]. In this project, this value is false, so we can override it to ensure our app appears online.

```javascript
// @ts-expect-error: navigator.onLine must be true
window.navigator["onLine"] = true;
```

## It works

Finally, restart your project, and it should work!

[^1]: [Use Firebase - Expo Documentation](https://docs.expo.dev/guides/using-firebase/#:~:text=services%20such%20as-,Authentication%2C%20Firestore%2C%20Realtime%20Database%2C%20and%20Storage,-in%20your%20app)
[^2]: [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
[^3]: [Does React Native support Indexed DB?](https://stackoverflow.com/a/52051320)
[^4]: [fakeIndexedDB](https://github.com/dumbmatter/fakeIndexedDB)
[^5]: [Non-ECMAScript JS APIs](https://github.com/facebook/hermes/discussions/1072)
[^6]: [Navigator: onLine property](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine)
