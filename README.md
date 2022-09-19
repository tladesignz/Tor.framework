# Tor.framework

[![Version](https://img.shields.io/cocoapods/v/Tor.svg?style=flat)](https://cocoapods.org/pods/Tor)
[![License](https://img.shields.io/cocoapods/l/Tor.svg?style=flat)](https://cocoapods.org/pods/Tor)
[![Platform](https://img.shields.io/cocoapods/p/Tor.svg?style=flat)](https://cocoapods.org/pods/Tor)

Tor.framework is the easiest way to embed Tor in your iOS application. The API is *not* stable yet, and subject to change.

Currently, the framework compiles in the following versions of `tor`, `libevent`, `openssl`, and `liblzma`:

|          |          |
|:-------- | --------:|
| tor      | 0.4.7.10 |
| libevent | 2.1.12   |
| OpenSSL  | 1.1.1q   |
| liblzma  | 5.2.6    |


## Example

To run the example project, clone the repo, and run `pod install` from the Example directory first.

## Requirements

- iOS 9.0 or later
- MacOS 12.0 or later
- Xcode 13.0 or later
- `autoconf`,  `automake`, `libtool` and  `gettext` in your `PATH`


## Installation

Install build tools via [Homebrew](https://brew.sh):

```sh
brew install automake autoconf libtool gettext
```

Tor is available through [CocoaPods](https://cocoapods.org). To install
it, simply add the following line to your Podfile:

If you use dynamic frameworks, use the root spec:

```ruby
use_frameworks!
pod 'Tor', '~> 407.10.1'
```

(or `Tor/GeoIP` - see below.)


If you need to add it as a static library, you will need to add it from a modified podspec:

```ruby
pod 'Tor', :podspec => 'https://raw.githubusercontent.com/iCepa/Tor.framework/v407.10.1/TorStatic.podspec'
```

Currently static library support is unstable. You might encounter build issues. 
Every contribution to fix this is welcome!

(or `Tor/GeoIP` - see below.)


## Preparing a new release

For maintainers/contributors of Tor.framework, a new release should be prepared by 
doing the following:

Ensure that you have committed changes to the submodule trees for tor, libevent, openssl, and xz.

Also update info and version numbers in `README.md` and `Tor.podspec`!

Then lint like this:

```sh
pod lib lint --verbose --allow-warnings
```

(Use `verbose`, otherwise you'll get *very* bored.)

If the linting went well, create a git tag for the version, push to GitHub and then publish to CocoaPods:

```sh
pod trunk push Tor.podspec --verbose --allow-warnings
```

(Unfortunately, you can not not lint on publish, so you might skip the first lint. However, `pod trunk push`
will take even longer, because it will clone everything fresh, too.)


Then create a [release](https://github.com/iCepa/Tor.framework/releases) in GitHub which corresponds
to the tag, and attach latest info as per older releases.


### Upgrading Tor

To upgrade Tor:

```bash
cd Tor/tor
git fetch
git checkout tor-0.4.7.10 # Find latest versions with git tag -l
rm -r * && git checkout . # Get rid of all autogenerated configuration files, which may not work with the newest version anymore.
git submodule update --init --recursive # Later Tor has submodules.
```

-> Test build by running the Example apps.

Check build output in the Report Navigator. (Last tab in the left pane.)


## Usage

Starting an instance of Tor involves using three classes: `TORThread`, `TORConfiguration` and `TORController`.

Here is an example of integrating Tor with `NSURLSession`:

```objc
TORConfiguration *configuration = [TORConfiguration new];
configuration.ignoreMissingTorrc = YES;
configuration.cookieAuthentication = YES;
configuration.dataDirectory = [NSURL fileURLWithPath:NSTemporaryDirectory()];
configuration.controlSocket = [configuration.dataDirectory URLByAppendingPathComponent:@"control_port"];

TORThread *thread = [[TORThread alloc] initWithConfiguration:configuration];
[thread start];

NSData *cookie = configuration.cookie;
TORController *controller = [[TORController alloc] initWithSocketURL:configuration.controlSocket];

NSError *error;
[controller connect:&error];

if (error) {
    NSLog(@"Error: %@", error);
    return;
}

[controller authenticateWithData:cookie completion:^(BOOL success, NSError *error) {
    if (!success)
        return;

    [controller addObserverForCircuitEstablished:^(BOOL established) {
        if (!established)
            return;

        [controller getSessionConfiguration:^(NSURLSessionConfiguration *configuration) {
            NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration];
            ...
        }];
    }];
}];
```


### GeoIP

In your `Podfile` use the subspec `GeoIP` or `StaticGeoIP` instead of the root spec:

```ruby
use_frameworks!
pod 'Tor/GeoIP'
```

or

```ruby
pod 'Tor/GeoIP', :podspec => 'https://raw.githubusercontent.com/iCepa/Tor.framework/pure_pod/TorStatic.podspec'
```

The subspec will create a "GeoIP" bundle and install a run script phase which 
will download the appropriate GeoIP files.

To use it with Tor, add this to your configuration:

```objc
TORConfiguration *configuration = [TORConfiguration new];
configuration.geoipFile = NSBundle.geoIpBundle.geoipFile;
configuration.geoip6File = NSBundle.geoIpBundle.geoip6File;
```

*ATTENTION: You might need to build two times to acquire the geoip files, due
to a limitation of CocoaPods!*


## Authors

Conrad Kramer, conrad@conradkramer.com
Chris Ballinger, chris@chatsecure.org
Mike Tigas. mike@tig.as
Benjamin Erhart, berhart@netzarchitekten.com


## License

Tor.framework is available under the MIT license. See the 
[`LICENSE`](https://github.com/iCepa/Tor.framework/blob/master/LICENSE) file for more info.
