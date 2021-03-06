# [fit]The Best Automated Tests
# [fit]You Don't Have to Write
### *Sean McMains -- @SeanMcTex*
### *Mutual Mobile*

---

# What's the Problem?

- Tests == Good
- Writing Tests == PITA
- Let's Min-Max this problem

^ Happy path, failing path, edge cases, sanity tests, etc. How to use open-source tools to maximize benefit:effort ratio

---

>> "People think computers will keep them from making mistakes. They're wrong. With computers you make mistakes faster." 
--Adam Osborne

---

# [FIT]Introducing KittenWeather

- "Because Kittens Make Everything Better"
- https://github.com/SeanMcTex/KittenWeather

![](http://khongthe.com/wallpapers/animals/adorable-kitten-104014.jpg)

^ Show app. Demonstrate features. But KittenWeather has some problems, and we're going to use some automated tools to find them.

---

# [FIT]Demo
# [fit]KittenWeather

---

# [fit]UIAutoMonkey

---

## UIAutoMonkey
- https://github.com/jonathanpenn/ui-auto-monkey
- written by Jonathan Penn, inspired by Netflix' [Chaos Monkey](https://github.com/Netflix/SimianArmy/wiki/Chaos-Monkey)
- Uses UIAutomation framework to simulate UI events
- Tests UI State, Stresses App

^ Chaos Monkey is a project from Netflix that randomly kills off systems in a cloud cluster. Only does it during business hours so folks are available to respond to and learn from the failures. Has helped make Netflix' infrastructure much more resiliant than it otherwise would have been.

![left](https://camo.githubusercontent.com/6a60562ae18dd21ab0156d3cbea37b4459ff57cf/68747470733a2f2f7261772e6769746875622e636f6d2f6a6f6e617468616e70656e6e2f75692d6175746f2d6d6f6e6b65792f6d61737465722f646f63696d672f6d6f6e6b65792e706e67)

---

# [fit]Demo

^ Load project in Xcode. CMD-I to launch Instruments. Choose Automation template. Make sure Script pane is selected. Add new script in Display Settings panel. Paste in the UIAutoMonkey.js file. Press Run. You can add additional tests later on if you want to monitor other aspects of how the app responds under stress.

---

# Fixing the problem
- Multiple copies of the About view provide a clue as to what's going on
- Add appropriate check to ViewController -didTapAbout;
- Rerun the monkey, verify that multiple about views and crash no longer occur.

---

# A Bit More On The Monkey

- Pretty heavily customizable. See options at top of .js file.
- Test double-taps!
- Because of randomness, tests aren't repeatable. Don't make this part of your unit test suite, but run it often.

^ Mention that it works fine with Swift, since it drives the UI.

---

# [fit]QuickCheck
# [fit]&
# [fit]Fox

---

# QuickCheck and Friends

* QuickCheck originated in Haskell; ported to lots of other languages
* Generates a slew of random test values, verifies that specified conditions hold true
* Write 1 test, get 499 for free
* Best library for Cocoa: Fox. https://github.com/jeffh/Fox

^ Fox supports Objective C and Swift, though the Swift docs are still pretty sketchy.

![left fit](http://www.clipartbest.com/cliparts/yco/ejr/ycoejrKMi.png)

---

# Setting up Fox

- Can be done either as a git submodule:

```
git submodule add https://github.com/jeffh/Fox.git Externals/Fox
```

- Or as a CocoaPods install
```
pod 'Fox', '~>1.0.1'	
```

---

# Write the Tests


```objectivec
- (void)testIdentityKelvin {
    id<FOXGenerator> kelvinTemperatureGenerator = FOXFloat();
    FOXAssert( FOXForAll(kelvinTemperatureGenerator, ^BOOL(NSNumber *kelvinTemperatureNumber) {
        Temperature *temperature = [Temperature 
			temperatureWithKelvinDegrees:[kelvinTemperatureNumber floatValue]];
        float difference = abs( [kelvinTemperatureNumber floatValue] - temperature.kelvinDegrees );
        if ( difference > 0.1 ) {
            return NO;
        } else {
            return  YES;
        }
    }));
}
```
		
^ My Temperature class needs some sanity checking. It supports temperatures in Imperial (Farenheit), Celsius, and Kelvin. And while it shouldn't be obvious from the public interface, we store the temperature internally in degrees Kelvin. (Show code.) Some of the most basic checks we can do on our Temperature class, then, are making sure that the number of degrees we put into an instance is the same as the number of degrees we get out when we query the object later. Here's what that test looks like for Kelvin temperatures. Since the code's a bit complex, let's break it down line by line.

---

# Create a Generator

```objectivec
id<FOXGenerator> kelvinTemperatureGenerator = FOXFloat();
```

^ First we instantiate an instance of a FOXFloat generator. Generators are the classes that Fox uses to create all of the random values that it will be throwing at your class. They also contain logic for shrinking the results when there's an error; we'll talk more about that later. Since temperatures can reasonably be represented as a float value, we use Fox's float generator here.

---
# Assert & Call Runner


```objectivec
FOXAssert( FOXForAll( kelvinTemperatureGenerator, 
	^BOOL( NSNumber *kelvinTemperatureNumber ) {
	...
}
```

^ Next we use FOXAssert to assert that the contained tests should pass. It takes a property assertion generator, of which there's currently only one available -- FOXForAll. FOXForAll is where the interesting stuff happens. We give it a generator -- the FOXFloat generator we've already defined in this case -- and a block that contains the actual test we'll be running. It's then repsonsible for using that generator to specify lots of 

---
# The Actual Test
### (What Does the Fox Say?)

```objectivec
Temperature *temperature = [Temperature 
	temperatureWithKelvinDegrees:[kelvinTemperatureNumber floatValue]];
float difference = fabs( [kelvinTemperatureNumber floatValue] - 
	temperature.kelvinDegrees );
if ( difference > 0.1 ) {
    return NO;
} else {
    return  YES;
}
```


---

#[fit]Demo

^ Note that the test for Imperial units is failing for very large values. This suggests a rounding error. Look at the error; note that it makes an effort to shrink to the smallest value that triggers the problem. Change math to use long doubles. Rerun tests, verify problem is now fixed.

---
# For Control Freaks

```objectivec
    FOXOptions options = {
        .seed=5,              // default: time(NULL)
        .numberOfTests=5000,  // default: 500
        .maximumSize=100,     // default: 200
    };
    FOXAssertWithOptions( FOXForAll(kelvinTemperatureGenerator, 
		^BOOL(NSNumber *kelvinTemperatureNumber) {
.		..
    }), options);
```

^ seed is used by the random number generator. Using the same seed makes tests reproducible. Number of tests determines how many values the generator will spit out. MaximumSize is a hint, used by different generators in different ways, as to how large the values it creates should be.

---

#[fit]Why You Want To Be
#[fit]A Control Freak

^ We want our tests to be reproducible. Nothing is more maddening than not being able to reproduce a failed test. Discuss random vs. pseudo-random. Since we're using a known algorithm for generating our pseudo-random numbers, we can get consistent results by specifying the options here.

---

# More About Fox

- Lots of generators, and you can make your own.
- Tutorial: http://fox-testing.readthedocs.org/en/latest/tutorial.html
- Using with Swift

---

#[fit]FBSnapshotTestCase

---
# FBSnapshotTestCase
- Testing UI is tough.
- Takes a baseline image of a UIView, compares it to image generated on subsequent runs.
- https://github.com/facebook/ios-snapshot-test-case

---

# Setup
- Use Pod

```		pod 'FBSnapshotTestCase'
```
- Add preprocessor definitions (optional)

```GCC_PREPROCESSOR_DEFINITIONS = $(inherited) FB_REFERENCE_IMAGE_DIR="\"$(SOURCE_ROOT)/$(PROJECT_NAME)Tests/ReferenceImages\""
```

---

# First Run (setting baselines)

```objectivec
	#import "FBSnapshotTestCase.h"
	...
	@interface ViewTests : FBSnapshotTestCase
	...
	-(void)setUp {
	    [super setUp]; // IMPORTANT!
	    self.recordMode = YES;
	    self.viewController = [[UIStoryboard storyboardWithName:@"Main" bundle:nil] 
			instantiateInitialViewController];
	}

	- (void)testRootView {
	    FBSnapshotVerifyView( self.viewController.view, nil );
	}
```

---

# Setting Things Up
```objectivec
	-(void)setUp {
	    [super setUp]; // IMPORTANT!
	    self.recordMode = YES;
	    self.viewController = [[UIStoryboard storyboardWithName:@"Main" bundle:nil] 
			instantiateInitialViewController];
	}
```

---

# Running the Test
```objectivec
	- (void)testRootView {
	    FBSnapshotVerifyView( self.viewController.view, nil );
	}
```

^ The second parameter is an optional identifier. You can usually leave this out, but if you have multiple calls to FBSnapshotVerifyView in a single test, you'll need to use this to differentiate them.

---

#[fit]Demo

^ Change placeholder text in IB. Rerun tests. Note test has failed. Look at log. Demonstrate Kaleidescope call. Demonstrate Xcode plugin . If you determine the new snapshot looks good, simply reset the baseline.

---

# A Few More Things

- Can be pointed at any UIView.
- Use CI to check different devices
- http://www.objc.io/issue-15/snapshot-testing.html has more details
- Xcode plugin: https://github.com/orta/snapshots
- Using with Swift

^ Setting this up to run automatically on different device sizes is possible, but is something of a headache.

---

# Summary

- __UIAutoMonkey__: UI State/Stress Testing
- __QuickCheck/Fox__: Randomized Automatic Validation Testing
- __FBSnapshotTestCase__: Snapshot Testing

---

# Conclusion

- Even if you don't go full-in on TDD or exhaustive unit testing, you can get a lot of value from automated tests.
- Look for tools that will maximize the benefit to effort ratio for your particular circumstances.

---

>> "If you don’t like testing your product, most likely your customers won’t like to test it either."

---

# Thank You

- https://github.com/SeanMcTex/KittenWeather
- Sean McMains -- @SeanMcTex
- sean.mcmains@mutualmobile.com
- http://www.mutualmobile.com/resources

