# [`matcher-core`](/): ORB-focused pattern-miner for PublicLab

![LICENSE](https://img.shields.io/badge/license-BSD--2--Clause-green.svg)

## Description

`matcher-core` essentially employs the ORB[(Oriented FAST and Rotated BRIEF)](http://www.willowgarage.com/sites/default/files/orb_final.pdf) algorithm to mine patterns using the well-known [FAST(Features from Accelerated Segment Test)](http://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/AV1011/AV1FeaturefromAcceleratedSegmentTest.pdf) keypoint detector and the [BRIEF(Binary Robust Independent Elementary Features)](https://www.cs.ubc.ca/~lowe/525/papers/calonder_eccv10.pdf) descriptor technologies, which provide appreciable performance boosts on low computational costs. The main advantages, without going in too deep into details, of building this module around [ORB]() were as follows.

	* About 10^2 times faster than [SURF(Speeded-Up Robust Features)](https://www.vision.ee.ethz.ch/~surf/eccv06.pdf), a close alternative before [ORB]() till 2011.
	* Addition of a fast and accurate orientation component to [FAST]().
	* Efficient in-built computational support for analysis of variance and correlation.
	* Decorrelates [BRIEF]() features under rotational invariance, leading to better performance in nearest-neighbor applications.
	* Unlike [SURF]() and [SIFT(Scale-Invariant Feature Transform)](http://weitz.de/sift/), which are patented algorithms, [ORB]() is free to use.

## Setup

* Include the browerified [`orb.core.com.js`](/orb.core.com.js) file inside your [`index.html`](/demo/index.html) using `<script>` tags.
```html
<script src="../orb.core.com.js"></script>
```
* The matcher-core library's [entry point file]() will return a promise back into the injected scope. Therefore, one needs to resolve the response and passing it to desired function scope before using it.
```html
// Inside index.html
<script>

// ...

// Resolving the returned values
Promise.resolve(new orbify('/path/to/img1.jpg', '/path/to/img2.jpg', {
	browser: true,	// required for browser (non-node) environments
	params: {
		lap_thres: 35,
		eigen_thres: 40,
		blur_size: 4,
		matchThreshold: 50
	}
}).utils).then(function (utils) {
	callback(utils);	// passing resolved value to callback's scope
});

// ...

</script>
```
The implementation snippet above was taken from [here](/demo/index.html).
* `matcher-core`'s `orbify` will accept the following format of input parameters.
```js
/* Object here represents the `args` object
* which consists of a "browser" field, and
* an additional "params" object that specifies
* different image filters' limits and
* thresholds, as specified in the example
* above.
*/
const instance = new orbify(<Image>, <Image>, <Object>);
```
* Similarly, `matcher-core`'s `orbify` will output the following data.
```js
/* returns a set of detected corners
* and the set of 'rich' matches that
* are evaluated from it
*/
{matches: Array(9), corners: Array(500)}
// which are formatted as depicted below
{
    "matches": [
        {
            "confidence": {
                "c1": 63,
                "c2": 187
            },
            "x1": 359,
            "y1": 48,
            "x2": 65,
            "y2": 309,
            "population": 9
				},
				...
		],
		"corners": [
        {
            "x": 37,
            "y": 261
				},
				...
		]
}
```
**Note:** The coordinates returned above are respective of the **image-pixel space**, hence are independent of their surrounding canvas spaces. In simpler terms, these coordinates actually represent the pixel numbers (*of images in their own x-y spaces*) on both axes in an image, wherever a point of interest is found.

## Demonstration

The live-demonstration of an [example file](/demo/index.html) using this library can be found on the `gh-pages` branch deployed [here](https://rexagod.github.io/matcher-core/demo/).

## Building from source

* To build modified source files, do:
	* `npm i -g browserify` to globally install [browserify](https://www.npmjs.com/package/browserify).
	* `browserify src -o orb.core.com.js` to build from the [`/src`](/src) directory.
	* Use the newly browserified [`orb.com.core.js`](/orb.com.core.js) file as the entry point.

## Codeflow

Below is a summary of each component of the [`orbify`](/orb.core.com.js#L14) instance, which is the at the core of this library, that returns a promise, and this should be kept in mind while extending the repository into one's own.

* [`canvas Mock`](/orb.core.com.js#L15-L28): Sets up two mock canvases for image comparison, which are never really appended to the DOM tree, along with some global variables.

* [`core IIFE`](/orb.core.com.js#L29-L191): Initializes `matchT` structure, `demoOpt` trainer, `demoApp` scraper, and the `tick` filters. These methods are detailed below.

* [`matchT structure`](/orb.core.com.js#L35-L55): An IIFE that initializes four basic parameters, namely, `screenIdx`, `patternLev`, `patternIdx`, and `distance` for the comparision structure. These are used later on in the `orbify` function to store the array of corners in the first image, number of discernable levels in the second image, array of corners in the second image, and the distance between the two indices, respectively. The default values for all of these, is set to 0.

* [`demoOpt trainer`](/orb.core.com.js#L57-L119): Initializes four adjustable filters namely, `blur_size`, `lap_thres`, `eigen_thres`, and `matchThreshold`, along with the `train_pattern` trainer function. The functions of these parameters *respectively* are as follows. 

	* Adjusting the blur on images to incorporate more corners for better detection or reducing them to focus on the dominant matches only.
	* Defining the rate of change of intensity of the pixels for them to be perceived as "noisy".
	* Specifying how similar two pixels should be in order to be perceived as "lying in the same cluster".
	* Specifying matching standards (measure of depth that the match instensity should be in order for it to be a good match).

* [`demoApp scraper`](/orb.core.com.js#L121-L142): Initializes `blurred U8` matrices, descriptors for both of the images, and a homographic matrix for 2D distance conversions, along with the [corners and matches (set of matchT structs, not to be confused with matches global variable)](/orb.core.com.js#L139-L140) non-primitive structure arrays.

* [`tick filters`](/orb.core.com.js#L144-L176): Draws out the images (from the canvas data), applies a set of filters, and calculates the number of `goodMatches`, which will be exported out as matches array, and will consist of all the rich matches from both of the pixel arrays (images). The set of filters mentioned above are as follows.

	* [`grayscale`](/orb.core.com.js#L154): Reduces noise (outliers) by blurring the modified input `img_u8` via 8-bit conversions.
	* [`gaussian_blur`](/orb.core.com.js#L156):Converting every pixel to a 0 (black) or 1 (white) value for increased performace and evaluation processes.
	* [`laplacian_threshold`](/orb.core.com.js#L158): (as above) Defines the rate of change of intensity of the pixels that should be perceived as "noisy".
	* [`eigen_value_threshold`](/orb.core.com.js#L159): (as above) Specify how similar two pixels should be in order for them to be perceived as "lying in the same cluster".

All this being said, if you still have any questions regarding `matcher-core`'s implementations, feel free to [open an issue](/issues) clearly specifying your doubts, and pinging me ([@rexagod](https://github.com/rexagod)) in the issue discription.

## LICENSE

[BSD-2-Clause](/LICENSE)