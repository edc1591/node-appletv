# node-appletv

> A node module for interacting with an Apple TV (4th-generation or later) over the Media Remote Protocol.

[![npm version](https://badge.fury.io/js/node-appletv.svg)](https://badge.fury.io/js/node-appletv)
[![CI](https://travis-ci.org/edc1591/node-appletv.svg?branch=master)](https://travis-ci.org/edc1591/node-appletv)
[![License][license-image]][license-url]

![](images/pairing.gif)

## Overview

`node-appletv` is a `node.js` implementation of the Media Remote Protocol which shipped with the 4th-generation Apple TV. This is the protocol that the Apple TV remote app uses, so this should enable the creation of an Apple TV remote app for various platforms. It can also be used in a `homebridge` plugin to connect Apple TV events to HomeKit and vice versa. `node-appletv` can be used as a standalone command line application, or as a module in your own node app. Keep reading for installation and usage instructions.

## Documentation

Developer documentation for `node-appletv` can be found [here](https://edc1591.github.io/node-appletv/).

## Usage

### As a standalone cli

```bash
# Install
$ npm install -g node-appletv

# Display built-in help
$ appletv --help
```

**Note** If you're getting an installation error such as below:
```bash
> sodium@2.0.3 preinstall /usr/lib/node_modules/node-appletv/node_modules/sodium
> node install.js --preinstall

Static libsodium was not found at /usr/lib/node_modules/node-appletv/node_modules/sodium/deps/build/lib/libsodium so compiling libsodium from source.
libtoolize:   error: Failed to create 'build-aux'
make: *** [libsodium] Error 1
Makefile:61: recipe for target 'libsodium' failed
/usr/lib/node_modules/node-appletv/node_modules/sodium/install.js:287
            throw new Error(cmdLine + ' exited with code ' + code);
            ^

Error: make libsodium exited with code 2
    at ChildProcess.<anonymous> (/usr/lib/node_modules/node-appletv/node_modules/sodium/install.js:287:19)
    at ChildProcess.emit (events.js:180:13)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:209:12)
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! sodium@2.0.3 preinstall: `node install.js --preinstall`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the sodium@2.0.3 preinstall script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /root/.npm/_logs/2018-05-10T04_39_57_013Z-debug.log
```
You can workaround the issue by installing with the following command:
```bash
sudo npm install -g node-appletv --unsafe-perm
```

The `appletv` cli supports several commands, such as:

`pair`: Scans for Apple TVs on the local network and initiates the pairing process

`command <command>`: Execute a command on an Apple TV (play, pause, menu, etc.)

`state`: Logs state changes from an Apple TV (now playing info)

`queue`: Requests the current playback queue from an Apple TV

`messages`: Logs all raw messages from an Apple TV

`help <command>`: Get help for a specific command


### As a node module

```bash
$ npm install --save node-appletv
```

`node-appletv` makes heavy use of Promises. All functions, except for the observe functions, return Promises.

### Examples

#### Scan for Apple TVs and pair

```typescript
import { scan } from 'node-appletv';

return scan()
    .then(devices => {
    	// devices is an array of AppleTV objects
    	let device = devices[0];
    	return device.openConnection()
    		.then(device => {
    			return device.pair();
    		})
    		.then(callback => {
    			// the pin is provided onscreen from the Apple TV
    			return callback(pin);
    		});
    })
    .then(device => {
    	// you're paired!
    	let credentials = device.credentials.toString();
    	console.log(credentials);
    })
    .catch(error => {
    	console.log(error);
    });
```

#### Connect to a paired Apple TV

```typescript
import { scan, parseCredentials, NowPlayingInfo } from 'node-appletv';

// see example above for how to get the credentials string
let credentials = parseCredentials(credentialsString);

return scan(uniqueIdentifier)
    .then(devices => {
    	let device = devices[0];
    	return device.openConnection(credentials);
    })
    .then(device => {
    	// you're connected!
    	// press menu
    	return device.sendKeyCommand(AppleTV.Key.Menu);
    })
    .then(device => {
    	console.log("Sent a menu command!");
    	
    	// monitor now playing info
    	device.on('nowPlaying', (info: NowPlayingInfo) => {
    		console.log(info.toString());
    	});
    })
    .catch(error => {
    	console.log(error);
    });
```

The `uniqueIdentifier` is advertised by each Apple TV via Bonjour. Use an app like [Bonjour Browser](http://www.tildesoft.com) to find it. The identifier is also the first value in the string value of the `Credentials` object.

See [homebridge-theater-mode](https://github.com/edc1591/homebridge-theater-mode) for a more practical use of this module.

## TODO

- [ ] Add wake command
- [ ] Save credentials for paired Apple TVs

## Development

`node-appletv` is written in Typescript. Edit files in the `src` directory and then run `npm link` to clean, build, and create the symlinks to use the library and cli.

## Acknowledgments

`node-appletv` would not have been possible without the work of these people:

* [Jean Regisser](https://github.com/jeanregisser) who reversed the protobuf [spec of the MediaRemoteTV protocol](https://github.com/jeanregisser/mediaremotetv-protocol)
* [Pierre Ståhl](https://github.com/postlund) who [implemented the protocol in Python](https://github.com/postlund/pyatv)
* [Khaos Tian](https://github.com/KhaosT) for [reversing the HomeKit protocol](https://github.com/KhaosT/HAP-NodeJS) which also uses SRP encryption
* [Zach Bean](https://github.com/forty2) for [implementing the HAP client spec](https://github.com/forty2/hap-client)

## Meta

You can find me on Twitter [@edc1591](https://twitter.com/edc1591)

Distributed under the MIT license. See ``LICENSE`` for more information.

[license-image]: https://img.shields.io/badge/License-MIT-blue.svg
[license-url]: LICENSE
