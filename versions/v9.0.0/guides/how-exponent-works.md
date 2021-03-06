---
title: How Expo Works
old_permalink: /versions/v9.0.0/guides/how-exponent-works.html
previous___FILE: ./building-standalone-apps.md
next___FILE: ./upgrading-exponent.md
---

While it's certainly not necessary to know any of this to use Expo, many engineers like to know how their tools work. We'll walk through a few key concepts here. You can also browse the source, fork, hack on and contribute to the Expo tooling on [github/@exponent](http://github.com/exponent).

## Opening an app from Expo in development

There are two user facing pieces here: the Expo app and the Expo development tool (either XDE or exp CLI). We'll just assume XDE here for simplicity of naming. When you open an app up in XDE, it spawns and manages two server processes in the background: the Expo Development Server and the React Native Packager Server.

![](./fetch-app-from-xde.png)

> **Note:** XDE also spawns a tunnel process, which allows devices outside of your LAN to access the the above servers without you needing to change your firewall settings. If you want to learn more, see [ngrok](https://ngrok.com/).

### `Expo Development Server`

This server is the endpoint that you hit first when you type the URL into the Expo app. Its purpose is to serve the **Expo Manifest** and provide a communication layer between the XDE UI and the Expo app on your phone or simulator.

#### `Expo Manifest`

The following is an example of a manifest being served through XDE. The first thing that you should notice is there are a lot of identical fields to `exp.json` (see the [Configuration with exp.json](configuration.html#exp) section if you haven't read it yet). These fields are taken directly from that file -- this is how the Expo app accesses your configuration.

```javascript
{
  "name":"My New Project",
  "description":"A starter template",
  "slug":"my-new-project",
  "sdkVersion":"8.0.0",
  "version":"1.0.0",
  "orientation":"portrait",
  "primaryColor":"#cccccc",
  "iconUrl":"https://s3.amazonaws.com/exp-brand-assets/ExponentEmptyManifest_192.png",
  "notification":{
    "iconUrl":"https://s3.amazonaws.com/exp-us-standard/placeholder-push-icon.png",
    "color":"#000000"
  },
  "loading":{
    "iconUrl":"https://s3.amazonaws.com/exp-brand-assets/ExponentEmptyManifest_192.png"
  },
  "entryPoint": "main.js",
  "packagerOpts":{
    "hostType":"tunnel",
    "dev":false,
    "strict":false,
    "minify":false,
    "urlType":"exp",
    "urlRandomness":"2v-w3z",
    "lanType":"ip"
  },
  "xde":true,
  "developer":{
    "tool":"xde"
  },
  "bundleUrl":"http://packager.2v-w3z.notbrent.internal.exp.direct:80/apps/new-project-template/main.bundle?platform=ios&dev=false&strict=false&minify=false&hot=false&includeAssetFileHashes=true",
  "debuggerHost":"packager.2v-w3z.notbrent.internal.exp.direct:80",
  "mainModuleName":"main",
  "logUrl":"http://2v-w3z.notbrent.internal.exp.direct:80/logs"
}
```

Every field in the manifest is some configuration option that tells Expo what it needs to know to run your app. The app fetches the manifest first and uses it to show your app's loading icon that you specified in `exp.json`, then proceeds to fetch your app's JavaScript at the given `bundleUrl` -- this URL points to the React Native Packager Server.

In order to stream logs to XDE, the Expo SDK intercepts calls to `console.log`, `console.warn`, etc. and posts them to the `logUrl` specified in the manifest. This endpoint is on the Expo Development Server.

### React Native Packager Server

If you use React Native without Expo, you would start the packager by running `react-native start` in your project directory. Expo starts this up for you and pipes `STDOUT` to XDE. This server has two purposes.

The first is to serve your app JavaScript compiled into a single file and translating any JavaScript code that you wrote which isn't compatible with your phone's JavaScript engine. JSX, for example, is not valid JavaScript -- it is a language extension that makes working with React components more pleasant and it compiles down into plain function calls -- so `<HelloWorld />` would become `React.createElement(HelloWorld, {}, null)` (see [JSX in Depth](https://facebook.github.io/react/docs/jsx-in-depth.html) for more information). Other language features like [async/await](https://blog.getexponent.com/react-native-meets-async-functions-3e6f81111173#.4c2517o5m) are not yet available in most engines and so they need to be compiled down into JavaScript code that will run on your phone's JavaScript engine, JavaScript Core.

The second purpose is to serve assets. When you include an image in your app, you will use syntax like `<Image source={require('./assets/example.png')} />`, unless you have already cached that asset you will see a request in the XDE logs like: `<START> processing asset request my-proejct/assets/example@3x.png`. Notice that it serves up the correct asset for the your screen DPI, assuming that it exists.

## Deployment

Deployment for Expo means compiling your JavaScript bundle with production flags enabled (minify, disable runtime development checks) and upload it along with any assets that it requires (see [Assets](assets.html#all-about-assets)) to CloudFront. We upload your `exp.json` configuration to our server. As soon as the publish is complete, users will receive the new version next time they open the app or refresh it, provided that they have a version of the Expo client that supports the `sdkVersion` specified in your `exp.json`.

> **Note:** To package your app for deployment on the Apple App Store or Google Play Store, see [Building Standalone Apps](building-standalone-apps.html#building-standalone-apps). Each time you update the SDK version you will need to rebuild your binary.

## SDK Versions

The `sdkVersion` of an Expo app indicates what version of the compiled ObjC/Java/C layer of Expo to use. Each `sdkVersion` roughly corresponds to a release of React Native plus the Expo libraries in the SDK section of these docs.

The Expo client app supports many versions of the Expo SDK, but an app can only use one at a time. This allows you to publish your app today and still have it work a year from now without any changes, even if we have completely revamped or removed an API your app depends on in a new version. This is possible because your app will always be running against the same compiled code as the day that you published it.

If you publish an update to your app with a new `sdkVersion`, if a user has yet to update to the latest Expo client then they will still be able to use the previous `sdkVersion`.

> **Note:** It's likely that eventually we will formulate a policy for how long we want to keep around sdkVersions and begin pruning very old versions of the sdk from the client, but until we do that, everything will remain backwards compatible.

## Opening a deployed Expo app

The process is essentially the same as opening an Expo app in development, only now we hit the Expo backend to get the manifest, and manifest points us to your app's JavaScript which now lives on CloudFront.

![](./fetch-app-production.png)

## Standalone apps

We also refer to these as "shell apps", because they are essentially a modified version of the Expo client that always points to a single published URL and does not include the Expo home screen.
