## Essential Image Optimisation

Addy Osmani. 

***

**Everyone should be automating their image compression.**

In 2017, if you're hand-tuning images, you're doing it wrong. It's easy to forget, best practices change and content that doesn't go through a build pipeline can easily slip.
To automate: Use [imagemin](https://github.com/imagemin/imagemin) or [libvps](https://github.com/jcupitt/libvips) for your build process. Many alternatives exist. 

Most CDNs (e.g [Akamai](https://www.akamai.com/us/en/solutions/why-akamai/image-management.jsp) and third-party solutions like [Cloudinary](https://cloudinary.com), [imgix](https://imgix.com), [Fastly's Image Optimizer](https://www.fastly.com/io/) or [ImageOptim API](https://imageoptim.com/api) offer comprehensive automated image optimization solutions.

The amount of time you'll spend reading blog posts and tweaking your config is >> the monthly fee for a service (Cloudinary has a [free](http://cloudinary.com/pricing) tier!). If you don't want to outsource this work for cost or latency concerns, the open-source options above are solid. Projects like [Imageflow](https://github.com/imazen/imageflow) or [Thumbor](https://github.com/thumbor/thumbor) enable self-hosted alternatives.

**Everyone should be compressing their images efficiently.**

At minimum: run your JPEGs through [MozJPEG](https://github.com/mozilla/mozjpeg) (`q=80` or lower is fine for web content) & consider [Progressive JPEG](http://cloudinary.com/blog/progressive_jpegs_and_green_martians) support, PNGs through [pngquant](https://pngquant.org/) and SVGs through [SVGO](https://github.com/svg/svgo). Instead of crazy huge animated GIFs, deliver H.264 videos (or WebM for Chrome, Firefox and Opera)! If you can't at least use [Giflossy](https://github.com/pornel/giflossy). 
If you can spare the extra CPU cycles, need higher-than-web-average quality & are okay with slow encode times: try [Guetzli](https://research.googleblog.com/2017/03/announcing-guetzli-new-open-source-jpeg.html). 

Some browsers advertise support for image formats via the Accept request header. This can be used to conditionally serve formats: e.g lossy [WebP](https://developers.google.com/speed/webp/) for Blink-based browsers like Chrome and fallbacks like JPEG/PNG for other browsers.

There's always more you can do. Tools exists to generate and serve srcset breakpoints. Resource selection can be automated with [client-hints](https://developers.google.com/web/updates/2015/09/automating-resource-selection-with-client-hints) and you can ship fewer bytes to users who opted into "data savings" in the browser using the [Save-Data](https://developers.google.com/web/updates/2016/02/save-data) hint.
***


The smaller in file-size you can make your images, the better a network experience you can offer your users - especially on mobile. In this write-up, we'll look at ways to reduce image size through modern compression techniques with minimal impact to quality. 

## Introduction

**Images are still the number one cause of bloat on the web.**

Images take up massive amounts of internet bandwidth because they often have large file sizes. According to the [HTTP Archive](http://httparchive.org/), 60% of the data transferred to fetch a web page is images composed of JPEGs, PNGs and GIFs. Images now account for [1.7MB](http://httparchive.org/interesting.php#bytesperpage) of the content loaded for the average site and 45% of these image requests are JPEGs. 

![](images/Modern-Image00.png "image_tooltip")

Per [Soasta and Google research](https://www.thinkwithgoogle.com/marketing-resources/experience-design/mobile-page-speed-load-time/) in 2016, images were the 2nd highest predictor of conversions. Sessions converting users had 38% fewer images.

Image optimisation consists of different measures that can reduce the filesize of your images. It ultimately depends on what visual fidelity your images require.

![](images/image-optimisation.jpg "image_tooltip")


Caption: Common image optimisations include compression, responsively serving them down based on screen size using [<picture>](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture) and resizing them to reduce image decode costs. 

<aside class="key-point"><b>Note:</b> If nothing else, use [ImageOptim](https://imageoptim.com/). It can significantly reduce the size of images while preserving visual quality. It also supports MozJPEG and Guetzli to take advantage of newer optimising JPEG encoders. If you're a designer, there's a new [ImageOptim plugin for Sketch](https://github.com/ImageOptim/Sketch-plugin) that will optimize your assets on export. I've found it a huge time saver.</aside>


#### How can I tell if my images need to be optimized?

Perform a site audit through [WebPageTest.org](https://www.webpagetest.org/) and it will highlight opportunities to better optimize your images (see "Compress Images"). Clicking through will display a report with a list of images that can be compressed more efficiently.


![](images/Modern-Image1.jpg "image_tooltip")

![](images/Modern-Image2.jpg "image_tooltip")


[Lighthouse](https://developers.google.com/web/tools/lighthouse/) is a Chrome extension that can audit for performance best practices on your local machine. It includes audits for image optimisation and can make suggestions for images that could be compressed further or point out images that are off-screen and could be lazy-loaded:

![](images/Modern-Image3.jpg "image_tooltip")


As of Chrome 60, Lighthouse now powers the [Audits panel](https://developers.google.com/web/updates/2017/05/devtools-release-notes#lighthouse) in the Chrome DevTools:

![](images/Modern-Image4.jpg "image_tooltip")


You may also be familiar of other performance auditing tools like [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) or [Website Speed Test](https://webspeedtest.cloudinary.com/) by Cloudinary which includes a detailed image analysis audit. 


### The humble JPEG.

The [JPEG](https://en.wikipedia.org/wiki/JPEG) may well be the world's most widely used image format. As noted earlier, [45% of the images](http://httparchive.org/interesting.php) seen on sites crawled by HTTP Archive are JPEGs. Your phone, your digital SLR, that old webcam - everything pretty much supports this codec. It's also very old, dating all the way back to 1992 when it was first released. In that time, there's been an immense body of research done attempting to improve what it offers. 

JPEG is a lossy compression algorithm that discards information in order to save space and many of the efforts that came after it attempted to preserve visual fidelity while keeping file-sizes as low as possible.

**What image quality is acceptable for your use-case?**

Formats like JPEG are best suited for photographs or images with a number of color regions. Most optimisation tools will allow you to set what level of compression you're happy with; higher compression reduces file size but can introduce artifacts, halos or blocky degrading. 

![](images/Modern-Image5.jpg "image_tooltip")


When choosing what quality setting to opt for, consider what quality bucket your images fall into:


*   **Best quality** - when quality matters more than bandwidth. This may be because the image has high prominence in your design or is displayed at full resolution.
*   **Good quality** - when you care about shipping smaller file-sizes, but don't want to negatively impact image quality too much. Users still care about some level of image quality.
*   **Low quality** - when you care enough about bandwidth that image degradation is okay. These images are suitable for spotty/poor network conditions.
*   **Lowest quality** - bandwidth savings are paramount. Users want a decent experience but will accept a pretty degraded experience for the benefit of pages loading more quickly.

Next, let's talk about JPEG's compression modes as these can have a large impact on perceived performance.

<aside class="note"><b>Note:</b> It's possible that we sometimes overestimate the image quality that our users need. Image quality could be considered a deviation from an ideal, uncompressed source. It can also be subjective.</aside>


## JPEG compression modes

The JPEG image format has a number of different [compression modes](http://web.ece.ucdavis.edu/cerl/ReliableJPEG/Cung/jpeg.html). Three popular modes are baseline (sequential), lossless and Progressive JPEG (PJPEG). 


### What's the difference between a baseline/sequential JPEG and a Progressive JPEG?

Baseline JPEGs (the default for most image editing & optimisation tools) are encoded and decoded in a relatively simple manner: top to bottom. When baseline JPEGs load on slow or spotty connections, users see the top of the image with more of it revealed as the image loads. Lossless JPEGs are similar but have a smaller compression ratio.


![](images/Modern-Image6.jpg "image_tooltip")


Progressive JPEGs divide the image into a number of scans. The first scans show the image in a blurry or low-quality setting and following scans improve image quality. Think of this as "progressively" refining it. Each "scan" of an image adds an increasing level of detail. When combined this creates a full-quality image.


![](images/Modern-Image7.jpg "image_tooltip")


Caption: Baseline JPEGs load images from top to bottom. PJPEGs load from low-resolution (blurry) to high-resolution. A Cloudinary demo of [decoded in slow motion](http://res.cloudinary.com/jon/video/upload/non_progressive_vs_progressive_JPEG.mp4) is available too.

Lossless JPEG optimization can be achieved by removing EXIF data added by digital cameras or editors, optimizing an image's [Huffman tables](https://en.wikipedia.org/wiki/Huffman_coding) or rescanning the image. Tools like jpegtran achieve lossless compression by rearranging the compressed data without image degradation. jpegrescan, jpegoptim and mozjpeg (which we'll cover shortly) also support lossless JPEG compression.


### The advantages of Progressive JPEGs

The ability for PJPEGs to offer low-resolution "previews" of an image as it loads improves perceived user performance - users can feel like the image is loading faster compared to adaptive images. 

On slower 3G connections, this allows users to see (roughly) what's in an image when only part of the file has been sent down and make a call on whether to wait for it to fully load. This can be more pleasant than the top-to-bottom display of images offered by baseline JPEGs.

![](images/Modern-Image8.jpg "image_tooltip")


Caption:[ Facebook switched to PJPEG (for their iOS app)](https://code.facebook.com/posts/857662304298232/faster-photos-in-facebook-for-ios/) and saw a 10% reduction in data usage. They were able to show a good quality image 15% faster than previously, optimising perceived loading time. 

PJPEGs can improve compression, consuming [2-10% ](http://www.bookofspeed.com/chapter5.html) less bandwidth compared to baseline/simple JPEGs for images over 10KB. Their higher compression ratio is thanks to each scan in the JPEG being able to have its own dedicated optional [Huffman table](https://en.wikipedia.org/wiki/Huffman_coding). Modern JPEG encoders (e.g [libjpeg-turbo](http://libjpeg-turbo.virtualgl.org/), MozJPEG etc. take advantage of PJPEG's flexibility to pack data better. 

*Why do PJPEGs compress better? Similar [Discrete Cosine Transform](https://en.wikipedia.org/wiki/Discrete_cosine_transform) coefficients across more than one block end up being encoded together which can compress better than simple/baseline JPEGs whose blocks are encoded one at a time.*


### Who's using Progressive JPEGs in production?



*   [Twitter.com ships Progressive JPEGs](https://www.webpagetest.org/performance_optimization.php?test=170717_NQ_1K9P&run=2#compress_images) with a baseline of quality of 85%. They measured user perceived latency (time to first scan and overall load time) and found overall, PJPEGs were competitive at addressing their requirements for low file-sizes, acceptable transcode and decode times.
*   [Facebook ships Progressive JPEGs for their iOS app](https://code.facebook.com/posts/857662304298232/faster-photos-in-facebook-for-ios/). They found it reduced data-usage by 15% and enabling them to show a good quality image 15% faster. 
*   [Yelp switched to Progressive JPEGs](https://engineeringblog.yelp.com/2017/06/making-photos-smaller.html) and found it was part responsible for ~4.5% of their image size reduction savings. They also saved an extra 13.8% using MozJPEG


### The disadvantages of Progressive JPEGs

PJPEGs can be slower to decode than baseline JPEGs - sometimes taking 3x as long. On desktop machines with powerful CPUs this can be less of a concern, but is on underpowered mobile devices with limited resources. Displaying incomplete layers takes work as you're basically decoding the image multiple times. These multiple passes can eat CPU cycles. libjpeg published a [performance study](http://www.libjpeg-turbo.org/About/Mozjpeg) including where PJPEG spends time available.

Progressive JPEGs are also not *always* smaller. For very small images (like thumbnails), progressive JPEGs can be larger than their baseline counterparts. However for such small thumbnails, progressive rendering might not really offer as much value.

This means that when deciding whether or not to ship PJPEGs, you'll need to experiment and find the right balance of file-size, network latency and use of CPU cycles.

Note: PJPEGs (and all JPEGs) can sometimes be hardware decodable on mobile devices. It doesn't improve on RAM impact, but it can negate some of the CPU concerns. Not all Android devices have H/W support, but high end devices do, and so do all iOS devices.

Some users may consider progressive loading to be a disadvantage as it can become hard to tell when an image has completed loading. As this can vary heavily per audience, evaluate what makes sense for your own users.


### How do you create Progressive JPEGs?

Tools and libraries like [ImageMagick](https://www.imagemagick.org/), [libjpeg](http://libjpeg.sourceforge.net/), [jpegtran](http://jpegclub.org/jpegtran/),[ jpeg-recompress](http://jpegclub.org/jpegtran/) and [imagemin](https://github.com/imagemin/imagemin) support exporting Progressive JPEGs. If you have an existing image optimization pipeline, there's a good likelihood that adding progressive loading support could be straight-forward:

```js
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');

gulp.task('images', function () {
    return gulp.src('images/*.jpg')
        .pipe(imagemin({
            progressive: true

        }))
        .pipe(gulp.dest('dist'));       
});
```

Most image editing tools save images as Baseline JPEG files by default. 

![](images/photoshop.jpg "image_tooltip")

You can save any image you create in Photoshop in Progressive JPEG by going to File -> Export -> Save for Web (legacy) and then clicking on the Progressive option. Sketch also supports exporting Progressive JPEGs - export as JPG and check the 'Progressive' checkbox while saving your images.


### How far have we come from the JPEG?

**Here's the current state of image formats on the web:**

*tl;dr - there's a lot of fragmentation. You need to conditionally serve different formats to different browsers to take advantage of anything modern.*

![](images/format-comparison.jpg "image_tooltip")

*   **[JPEG 2000](https://en.wikipedia.org/wiki/JPEG_2000) (2000)** - an improvement to JPEG switching from a discrete cosine based transform to a wavelet-based method. **Browser support: Safari desktop + iOS**
*   **[JPEG XR](https://en.wikipedia.org/wiki/JPEG_XR) (2009)** - alternative to JPEG and JPEG 2000 supporting [HDR](http://wikivisually.com/wiki/High_dynamic_range_imaging) and wide [gamut](http://wikivisually.com/wiki/Gamut) color spaces. Produces smaller files than JPEG at slightly slower encode/decode speeds. **Browser support: Edge, IE.**
*   **[WebP](https://en.wikipedia.org/wiki/WebP) (2010)** - block-prediction based format by Google with support for lossy and lossless compression. Offers byte savings associated with JPEG and transparency support byte-heavy PNGs are often used for. Lacks chroma subsampling configuration and progressive loading. Decode times are also slower than JPEG decoding. **Browser support: Chrome, Opera. Experimented with by Safari and Firefox.**
*   **[FLIF](https://en.wikipedia.org/wiki/Free_Lossless_Image_Format) (2015)** - lossless image format claiming to outperform PNG, lossless WebP, lossless BPG and lossless JPEG 2000 based on compression ratio. **Browser support: none.**
*   **HEIF and BPG.** From a compression perspective, they're the same but have a different wrapper:
*   **[BPG](https://en.wikipedia.org/wiki/Better_Portable_Graphics) (2015)** - intended to be more compression-efficient replacement for JPEG, based on HEVC ([High Efficiency Video Coding](http://wikivisually.com/wiki/High_Efficiency_Video_Coding)). Appears to offer better file size compared to MozJPEG and WebP. Unlikely to get broad traction due to licensing issues. **Browser support: none. *Note that there is a [JS in-browser decoder](https://bellard.org/bpg/).***
*   **[HEIF](https://en.wikipedia.org/wiki/High_Efficiency_Image_File_Format) (2015)** - format for images and image sequences for storing HEVC-encoded images with constrained inter-prediction applied. Apple announced at [WWDC](https://www.cnet.com/news/apple-ios-boosts-heif-photos-over-jpeg-wwdc/) they would explore switching to HEIF over JPEG for iOS, citing up to 2x savings on file-size. **Browser support: None at the time of writing. Eventually, Safari desktop and iOS 11**

If you're more visual, you might appreciate [one](https://people.xiph.org/~xiphmont/demo/daala/update1-tool2b.shtml) of [these](http://xooyoozoo.github.io/yolo-octo-bugfixes/#cologne-cathedral&jpg=s&webp=s) visual comparison tools for some of the above. 

So, **browser support is fragmented** and if you wish to take advantage of any of the above you'll likely need to conditionally serve fallbacks for each of your target browsers. At Google, we've seen some promise with WebP so we'll dive into it in more depth shortly.

Next, let's talk about an option for when you can't conditionally serve different image formats: **optimising JPEG encoders**. 


### Optimising JPEG Encoders

Modern JPEG encoders attempt to produce smaller, higher fidelity JPEG files while maintaining compatibility with existing browsers and image processing apps. They avoid the need to introduce new image formats or changes in the ecosystem in order for compression gains to be possible. Two such options are MozJPEG and Guetzli.

***tl;dr Which optimising JPEG Encoder should you use?***

* General web assets: MozJPEG

* Quality is your key concern and you don't mind long encode times: use Guetzli

* If you need configurability: JPEGRecompress (which uses MozJPEG under the hood)

There's also: 

* [JPEGMini](http://www.jpegmini.com/). It's similar to Guetzli - chooses best quality automatically. It's not as technically sophisticated as Guetzli, but it's faster, and aims at quality range more suitable for the web.

* [ImageOptim API](https://imageoptim.com/api) (with free online interface here: https://imageoptim.com/online) - it's unique in its handling of color. You can choose color quality separately from overall quality. It automatically chooses chroma subsampling level to preserve high-res colors in screenshots, but avoid waste bytes on smooth colors in natural photos.


#### What is MozJPEG? 

Mozilla offers a modernized JPEG encoder in the form of [MozJPEG](https://github.com/mozilla/mozjpeg). It [claims](https://research.mozilla.org/2014/03/05/introducing-the-mozjpeg-project/) to shave up to 10% off JPEG files. Files compressed with MozJPEG work cross-browser and some of its features include progressive scan optimization, [trellis quantization](https://en.wikipedia.org/wiki/Trellis_quantization) (discarding details that compress the least) and a few decent [quantization table presets](https://calendar.perfplanet.com/2014/mozjpeg-3-0/) that help create smoother High-DPI images (although this is possible with ImageMagick if you're willing to wade through XML configs). 

MozJPEG is supported in both [ImageOptim](https://github.com/ImageOptim/ImageOptim/issues/45) and there's a configurable [imagemin plugin](https://github.com/imagemin/imagemin-mozjpeg) for it I've found relatively reliable. Here's a sample of how I use it with Gulp:

```js
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');
const imageminMozjpeg = require('imagemin-mozjpeg');

gulp.task('mozjpeg', () =>
    gulp.src('src/*.jpg')
    .pipe(imagemin([imageminMozjpeg({
        quality: 85

    })]))
    .pipe(gulp.dest('dist'))
);
```

![](images/Modern-Image10.jpg "image_tooltip")


![](images/Modern-Image11.jpg "image_tooltip")


I used jpeg-compress from the [jpeg-archive](https://github.com/danielgtaylor/jpeg-archive) project to calculate the SSIM (The Structural Similarity) scores for a source image. SSIM is a method for measuring the similarity between two images, where we score the quality of one image relative to another that is considered "perfect".

In my experience, MozJPEG is a good option for compressing images for the web at a high visual quality while delivering reductions on file size. For small to medium sized images, I found MozJPEG (at quality=80-85) led to 30-40% savings on file size while maintaining acceptable SSIM, offering a 5-6% improvement on jpeg-turbo. It does come with a [slower encoding cost](http://www.libjpeg-turbo.org/About/Mozjpeg) than baseline JPEG, but you may not find this a show stopper.

<aside class="note"><b>Note:</b> if you need a tool supporting MozJPEG with additional configuration support and some complimentary utilities for image comparison, check out [jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive). Jeremy Wagner, author of Web Performance in Action has had some success using it with [this](https://twitter.com/malchata/status/884836650563579904) configuration.</aside>


#### What is Guetzli?

[Guetzli](https://github.com/google/guetzli) is a promising, if slow, perceptual JPEG encoder from Google that tries to find the smallest JPEG that can't be easily distinguished from the original by the human eye. It performs a sequence of experiments that produces a proposal for the final JPEG, accounting for the psychovisual error of each proposal. Out of these, it selects the highest-scoring one as the final output.

For this phase, Guetzli uses [Butteraugli](https://github.com/google/butteraugli) (a model for measuring image difference based on human perception - more on that soon!) that can take into account a few properties of vision that other JPEG encoders. For example, there is a relationship between the amount of green light seen and sensitivity to blue so changes in blue in the vicinity of green can be encoded a little less precisely. 

<aside class="note"><b>Note:</b> Image file-size is* much* more dependent on the choice of *quality* than the choice of codec. There are far far larger file-size differences between the lowest quality JPEG than the highest quality JPEG - compared to the difference switching codecs. Using the lowest acceptable quality is very important. Avoid setting your quality too high without paying attention to it.</aside>

Guetzli [claims](https://research.googleblog.com/2017/03/announcing-guetzli-new-open-source-jpeg.html ) to achieve a 20-30% reduction in data-size for images for a given Butteraugli score compared to other compressors. A large caveat to using Guetzli is that it is extremely, extremely slow and is currently only suitable for static content. From the README, we can note Guetzli requires a large amount of memory - it can take 1 minute + 200MB RAM per megapixel. There's a good thread on real-world experience with what this practically means in [this Github thread](https://github.com/google/guetzli/issues/50).

Tools like ImageOptim support Guetzli optimization (in [the latest versions](https://imageoptim.com/)). 

```js
const gulp = require('gulp');

const imagemin = require('gulp-imagemin');
const imageminGuetzli = require('imagemin-guetzli');

gulp.task('guetzli', () =>
    gulp.src('src/*.jpg')
    .pipe(imagemin([
        imageminGuetzli({
            quality: 85
        })
    ]))
    .pipe(gulp.dest('dist'))

);
```


![](images/Modern-Image12.jpg "image_tooltip")


It took multiple minutes (and high CPU usage) to encode 3 x 3MP images using Guetzli with varied savings. For archiving higher-resolution photos, I could see this offering some value.


![](images/Modern-Image13.jpg "image_tooltip")


<aside class="note"><b>Note:</b> It's recommended to run Guetzli on high quality images (e.g uncompressed input images, PNG sources or JPEGs of 100% quality or close). While it will work on other images (e.g JPEGs of quality 84 or lower), results can be poorer.</aside>

In my experience, compressing an image with Guetzli is very (very) time-consuming and will make your fans spin. That said, for larger images, this was worth it. I saw plenty of examples where it saved anywhere up to 40% on filesize while maintaining visual fidelity. This made it perfect for archiving photos. On small to medium sized images, I still saw some savings (in the 10-15KB range) but they were not quite as well pronounced.


#### How does MozJPEG compare to Guetzli? 

Comparing different JPEG encoders is complex - one needs to compare both the quality and fidelity of the compressed image as well as the final size. As image compression expert Kornel Lesinski notes, benchmarking one but not both of these aspects could lead to [invalid](https://kornel.ski/faircomparison) conclusions. 

How does Guetzli compare to MozJPEG? - Kornel's take:

- Guetzli is tuned for higher-quality images (butteraugli is said to be best for q=90+, MozJPEG's sweet spot is around q=75)

- Guetzli is much slower to compress (both produce standard JPEGs, so decoding is fast as usual)

- MozJPEG doesn't automagically pick quality setting, but you can find optimal quality using an external tool, e.g. https://github.com/danielgtaylor/jpeg-archive

A number of methods exist for determining if compressed images are visually similar or perceivably similar to their sources. Image quality studies often use methods like [SSIM](https://en.wikipedia.org/wiki/Structural_similarity) (structural similarity). Guetzli however optimizes for Butteraugli.


#### Butteraugli

[Butteraugli](https://github.com/google/butteraugli) is a project by Google that estimates the point when a human might notice the point of image degradation (the psychovisual similarity) of two images. It gives a score for the images that is reliable in the domain of barely noticeable differences. Butteraugli not only gives a scalar score, but also computes a spatial map of the level of differences. While SSIM looks at the aggregate of errors from an image, Butteraugli looks at the worst part.


![](images/Modern-Image14.jpg "image_tooltip")


Caption: Above is an example that used Butteraugli to find the minimal JPEG quality threshold before visual degradation was bad enough that a user would be able to notice something wasn't that clear. It resulted in a 65% reduction in total file size.

In practice, you would define a target goal for visual quality and then run through a number of different image optimisation strategies, looking at your Butteraugli scores, before choosing something that fits the best balance of file- size and level.

![](images/Modern-Image15.jpg "image_tooltip")


Caption: All in all, it took me about 30m to setup Butteraugli locally after installing Bazel and getting a build of the C++ sources to correctly compile on my Mac. Using it is then relatively straight-forward - specify the two images to compare (a source and compressed version) - and it will give you a score to work off.

**How does Butteraugli differ to other ways of comparing visual similarity?**

[This comment](https://github.com/google/guetzli/issues/10#issuecomment-276295265) from a Guetzli project member suggests Guetzli scores best on Butteraugli, worst on SSIM and MozJPEG scores about as well on both. This is in line with the research I've put into my own image optimisation strategy. I run Butteraugli and a Node module like [img-ssim](https://www.npmjs.com/package/img-ssim) over images comparing the source to their SSIM scores before/after Guetzli and MozJPEG. 

**Combining encoders?**

For larger images, I found combining Guetzli with **lossless compression **in MozJPEG (jpegtran, not cjpeg to avoid throwing away the work done by Guetzli) can lead to a further 10-15% decrease in filesize (55% overall) with only very minor decreases in SSIM. This is something I would caution requires experimentation and analysis but has also been tried by others in the field like [Ariya Hidayat](ariya.io/2017/03/squeezing-jpeg-images-with-guetzli) with promising results.

**MozJPEG is a beginner-friendly encoder for web assets that is relatively fast and produces good-quality images. As Guetzli is resource-intensive and works best on larger, higher-quality images, it's an option I would reserve for intermediate to advanced users.**


### What is WebP? 

[WebP](https://developers.google.com/speed/webp/) is a recent image format from Google aiming to offer lower file-sizes for lossless and lossy compression at an acceptable visual quality. It includes support for alpha-channel transparency and animation.

In the last year, WebP gained a few percent compression-wise in lossy and lossless  modes and speed-wise the algorithm got twice as fast with a 10% improvement in decompression.  WebP is not a tool for all purposes, but it has some standing and a growing user base in the image compression community. Let's examine why.


![](images/Modern-Image16.jpg "image_tooltip")


### How Does WebP Perform? 

***Lossy Compression***

WebP lossy files, using a VP8 or VP9 video key frame encoding variant, are on average cited by the WebP team as being [25-34%](https://developers.google.com/speed/webp/docs/webp_study) smaller than JPEG files. 

In the low-quality range (0-50), WebP has a large advantage over JPEG because it can blur away ugly blockiness artifacts. A medium quality setting (-m 4 -q 75) is the default balancing speed/file-size. In the higher-range (80-99), the advantages of WebP shrink. WebP is recommended where speed matters more than quality. 

***Lossless Compression***

[WebP lossless files are 26% smaller than PNG files](https://developers.google.com/speed/webp/docs/webp_lossless_alpha_study). The lossless load-time decrease compared to PNG is 3%. That said, you generally don't want to deliver your users lossless on the web. There's a difference between lossless and sharp edges (e.g non-JPEG). Lossless WebP may be more suitable for archival content.

***Transparency***

WebP has a lossless 8-bit transparency channel with only 22% more bytes than PNG. It also supports lossy RGB transparency, which is a feature unique to WebP.

***Metadata***

The WebP file format supports EXIF photo metadata and XMP digital document metadata. It also contains an ICC Color Profile.

WebP offers better compression at the cost of being more CPU intensive. Back in 2013, the compression speed of WebP was ~10x slower than JPEG but is now negligible (some images may be 2x slower). For static images that are processed as part of your build, this shouldn't be a large issue. Dynamically generated images will likely cause a perceivable CPU overhead and will be something you will need to evaluate. 

<aside class="note"><b>Note:</b> WebP lossy quality settings are not directly comparable to JPEG. A JPEG at "70% quality" will be quite different to a WebP image at "70% quality" because WebP achieves smaller file sizes by discarding more data.</aside>


### Who's using WebP in production?

Many large companies are using WebP in production to reduce costs and decrease web page load times. Google, of course, uses it in most of its properties like Gmail and YouTube. Netflix, Amazon, Quora, Yahoo, Walmart, Ebay, The Guardian, Fortune, and USA Today, all compress and serve images with WebP for browsers which support it. VoxMedia [shaved 1-3s off load times](https://product.voxmedia.com/2015/8/13/9143805/performance-update-2-electric-boogaloo) for The Verge by switching over to WebP for their Chrome users. There are quite a few more companies on board than this sample list indicates. 

Google reported a 30-35% savings using WebP over other lossy compression schemes, serving 43 billion image requests a day, 26% of that being lossless compression. That's a lot of requests and significant savings. Savings would undoubtedly be larger if browser support were better and more widespread.

![](images/webp-conversion.jpg "image_tooltip")

### How does WebP encoding work? 

WebP [Compression Techniques](https://developers.google.com/speed/webp/docs/compression) goes into this topic in depth but let's summarize some highlights. WebP's lossy encoding is designed to compete with JPEG for still images. There are three key phases to WebP's lossy encoding:



1.  **Macro-blocking** - splitting an image into 16x16 (macro) blocks of luma pixels and two 8x8 blocks of chroma pixels. This may sound familiar to the idea of JPEGs doing color space conversion, chroma channel downsampling and image subdivision. 

![](images/Modern-Image18.png "image_tooltip")


2. **Prediction** - every 4x4 subblock of a macroblock has a prediction model applied that effectively does filtering. This defines two sets of pixels around a block - A (the row directly above it) and L (the column to the left of it). Using these two the encoder fills a test block with 4x4 pixels and determines which creates values closest to the original block. Colt McAnlis talks about this in more depth in [How WebP lossy mode works](https://medium.com/@duhroach/how-webp-works-lossly-mode-33bd2b1d0670).


![](images/Modern-Image19.png "image_tooltip")


3. A Discrete Cosine Transform (DCT) is applied with a few steps similar to JPEG encoding. A key difference is use of an [Arithmetic Compressor](https://www.youtube.com/watch?v=FdMoL3PzmSA&index=7&list=PLOU2XLYxmsIJGErt5rrCqaSGTMyyqNt2H) vs JPEG's Huffman.


### Browser Support

Not all browsers support WebP, however [according to CanIUse.com](http://caniuse.com/webp), global user support is at about 74%. Chrome and Opera natively support it. Safari, Edge, and Firefox have experimented with it but not landed it yet in official releases. This often leaves the task of getting the WebP image to the user up to the web developer. More on this later.

Here are the major browsers and support information for each:

* Chrome: Chrome began full support at version 23.
* Chrome for Android: Since Chrome 50
* Android: Since Android 4.2 
* Opera: Since 12.1
* Opera Mini: All versions
* Firefox: Some beta support
* Edge: Some beta support
* Internet Explorer: No support
* Safari: Some beta support

WebP is not without its downsides. It lacks full-resolution color space options and does not support progressive decoding. That said, tooling for WebP is decent and browser-support, while limited to Chrome and Opera at the time of writing may well cover enough of your users for it to be worth considering with a fallback.


### How Do I Convert My Images to WebP?

Several commercial and open source image editing and processing packages support WebP. One particularly useful application is XnConvert: a free, cross-platform, batch image processing converter.

<aside class="note"><b>Note:</b> It's important to avoid converting low or average quality JPEGs to WebP. This is a common mistake and can generate WebP images with JPEG compression artifacts. This can lead to WebP being less efficient as it has to save the image _and_ the distortions added by JPEG, leading to you losing on quality twice. Feed conversion apps the best quality source file available.</aside>

*[XnConvert](http://www.xnview.com/en/xnconvert/)*

XnConvert enables batch image processing, compatible with over 500 image formats. You can combine over 80 separate actions to transform or edit your images in multiple ways. 


![](images/Modern-Image20.jpg "image_tooltip")

![](images/Modern-Image21.jpg "image_tooltip")


Some of the options listed on the xnview website include:

*   Metadata: Editing
*   Transforms: Rotate, Crop, Resize, … 
*   Adjustments: Brightness, Contrast, Saturation, … 
*   Filters: Blur, Emboss, Sharpen, …
*   Effects: Masking, Watermark, Vignetting, …
The results of your operations can be exported to about 70 different file formats, including WebP. XnConvert is free for Linux, Mac, and Windows. XnConvert is highly recommended, especially for small businesses.

**Node modules**

[Imagemin](https://github.com/imagemin/imagemin) is a popular image minification module that also has an add-on for converting images to WebP ([imagemin-webp](https://github.com/imagemin/imagemin-webp)). This supports both lossy and lossless modes.

To install imagemin and imagemin-webp run:

```
> npm install --save imagemin imagemin-webp
```

We can then require() in both modules and run them over any images (e.g JPEGs) in a project directory. Below we're using lossy encoding with a WebP encoder quality of 60:


```js
const imagemin = require('imagemin');
const imageminWebp = require('imagemin-webp');

imagemin(['images/*.{jpg}'], 'images', {
    use: [
        imageminWebp({quality: 60})
    ]
}).then(() => {
    console.log('Images optimized');
});
```


Similar to JPEGs, it's possible to notice compression artefacts in our output. Evaluate what quality setting makes sense for your own images. Imagemin-webp can also be used to encode lossless quality WebP images (supporting 24-bit color and full transparency) by passing `lossless: true` to options:


```js
imagemin(['images/*.{jpg,png}'], 'build/images', {
    use: [
        imageminWebp({lossless: true})
    ]
}).then(() => {
    console.log('Images optimized');
});
```


A [WebP plugin for Gulp](https://github.com/sindresorhus/gulp-webp) by Sindre Sorhus built on imagemin-webp and a [WebP loader for WebPack](https://www.npmjs.com/package/webp-loader) are also available. The Gulp plugin accepts any options the imagemin add-on does:

```js
const gulp = require('gulp');
const webp = require('gulp-webp');

gulp.task('webp', () =>
    gulp.src('src/*.jpg')
    .pipe(webp({
        quality: 80,
        preset: 'photo',
        method: 6
    }))
    .pipe(gulp.dest('dist'))
);
```

Or lossless:

```js
const gulp = require('gulp');
const webp = require('gulp-webp');

gulp.task('webp-lossless', () =>
    gulp.src('src/*.jpg')
    .pipe(webp({
        lossless: true
    }))
    .pipe(gulp.dest('dist'))
);
```

**Batch image optimization using Bash**

XNConvert supports batch image compression, but if you would prefer to avoid using an app or a build system, bash and image optimization binaries keep things fairly simple. 

You can bulk convert your images to WebP using [cwebp](https://developers.google.com/speed/webp/docs/cwebp):

```
find ./ -type f -name '*.jpg' -exec cwebp -q 70 {} -o {}.webp \;
```

Or bulk optimize your image sources with MozJPEG using [jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive):

```
find ./ -type f -name '*.jpg' -exec jpeg-recompress {} {} \;
```

and trim those SVGs down using [svgo](https://github.com/svg/svgo) (which we'll cover later on):

```
find ./ -type f -name '*.svg' -exec svgo {} \;
```

Jeremy Wagner has a more comprehensive post on[ image optimization using Bash](https://jeremywagner.me/blog/bulk-image-optimization-in-bash) and another on doing this work in [parallel](https://jeremywagner.me/blog/faster-bulk-image-optimization-in-bash) worth reading.

**Other WebP image processing and editing apps include:**

   * Leptonica — An entire website of open source image processing and analysis 

Apps.



*   Sketch supports outputting directly to WebP

    * GIMP — Free, open source Photoshop alternative. Image editor.

    * ImageMagick — Create, compose, convert, or edit bitmap images. Free. Command-Line app.

    * Pixelmator — Commercial image editor for Mac.

    * Photoshop WebP Plugin — Free. Image import and export. From Google.

**Android:** You can convert existing BMP, JPG, PNG or static GIF images to WebP format using Android Studio. For more information, see [Create WebP Images Using Android Studio](https://developer.android.com/studio/write/convert-webp.html).


### How do I view WebP images on my OS?

Although you can drag and drop WebP images to Blink-based browsers (Chrome, Opera, Brave) to preview them, you can also preview them directly from your OS using an add-on for either Mac or Windows.

[Facebook experimented with WebP](https://www.cnet.com/news/facebook-tries-googles-webp-image-format-users-squawk/) a few years ago and found that users who tried to right-click on photos and save them to disk noticed they wouldn't be displayed outside their browser due to them being in WebP. This might matter less to your users, but is an interesting note on social shareability in passing.

On Mac, try the [Quick Look plugin for WebP](https://github.com/Nyx0uf/qlImageSize) (qlImageSize). It works pretty well:

![](images/Modern-Image22.jpg "image_tooltip")

On Windows, you can also download the [WebP codec package](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/WebpCodecSetup.exe) allowing WebP images to be previewed in the File Explorer and Windows Photo Viewer.  


#### How Do I Serve WebP?

Browsers without WebP support can end up not displaying an image at all, which isn't ideal. To avoid this there are a few strategies we can use for conditionally serving WebP based on browser support.


![](images/play-format-webp.jpg "image_tooltip")

![](images/play-format-type.jpg "image_tooltip")


Caption: The Play store conditionally serves WebP images to supported browsers and fallback to JPEGs for browsers like Firefox.

Here are some of the options for getting WebP images from your server to your user: 

*Using .htaccess to Serve WebP Copies*

Here's how to use a .htaccess file to serve WebP files to supported browsers when a matching .webp version of a JPEG/PNG file exists on the server.

Vincent Orback recommended this approach:

Browsers with WebP support signal their support explicitly in the Accept header. If you happen to control your backend, you can just return the WebP version of an image if it exists on disk instead of the JPEG or PNG. This, however, isn't always possible if using a static host like S3 or GitHub pages.

Here is a sample .htaccess file for the Apache web server:

```
<IfModule mod_rewrite.c>

  RewriteEngine On

  # Check if browser support WebP images

  RewriteCond %{HTTP_ACCEPT} image/webp

  # Check if WebP replacement image exists

  RewriteCond %{DOCUMENT_ROOT}/$1.webp -f

  # Serve WebP image instead

  RewriteRule (.+)\.(jpe?g|png)$ $1.webp [T=image/webp,E=accept:1]

</IfModule>

<IfModule mod_headers.c>

    Header append Vary Accept env=REDIRECT_accept

</IfModule>

AddType  image/webp .webp
```

If there are issues with the .webp images appearing on the page, make sure that the image/webp MIME type is enabled on your server. 

Apache: add the following code to your .htaccess file:

```
AddType image/webp .webp
```

Nginx: add the following code to your mime.types file:

```
image/webp webp;
```

https://github.com/vincentorback/WebP-images-with-htaccess

***Using the <picture> Tag***

The browser itself is capable of choosing which image format to display through the use of the `<picture>` tag. The `<picture>` tag utilizes multiple <source> elements, with one `<img>` tag, which is the actual DOM element which contains the image. The browser cycles through the sources and retrieves the first match. If the `<picture>` tag isn't supported in the user's browser, a `<div>` is rendered and the `<img>` tag is used. 

<aside class="note"><b>Note:</b> Be careful with the position of `<source>` as order matters. Don't place image/webp sources after legacy formats, but instead put them before. Browsers that understand it will use them and those that don't will move onto more widely supported frameworks.</aside>

Here is some sample HTML: 

```html
<picture>
  <source srcset="/path/to/image.webp" type="image/webp">
  <img src="/path/to/image.jpg" alt="">
</picture>

<picture>   
    <source srcset='paul_irish.jxr' type='image/vnd.ms-photo'>  
    <source srcset='paul_irish.jp2' type='image/jp2'> 
    <source srcset='paul_irish.webp' type='image/webp'>
    <img srcset='paul_irish.jpg' alt='paul'>
</picture>

<picture>
   <source srcset="photo.jxr" type="image/vnd.ms-photo">
   <source srcset="photo.jp2" type="image/jp2">
   <source srcset="photo.webp" type="image/webp">
   <img srcset="photo.jpg" alt="My beautiful face">
</picture>
```

**Automatic CDN Conversion**

Some CDNs automatically perform browser sniffing and serve up WebP images whenever possible. Check with your CDN to see if WebP support is included in their service. You may have an easy solution just waiting for implementation.

**WordPress WebP Support**

Jetpack — Jetpack, a popular WordPress plugin, includes a CDN image service called Photon. With Photon you get seamless WebP image support. The Photon CDN is included in Jetpack's free level, so this is a good value and a hands-off implementation. The drawback is that Photon resizes your image, puts a query string in your URL and there is an extra DNS lookup required for each image.

**Cache Enabler and Optimizer** — If you are using WordPress, there is at least one halfway-open source option. The open source plugin Cache Enabler has a menu checkbox option for caching WebP images to be served if available and the current user's browser supports them. This makes serving WebP images easy. There is a drawback. Cache Enabler requires the use of a sister program called Optimizer, which costs at least $19 USD per year if you want to compress compatible WebP images servable with Cache Enabler. While inexpensive, this seems out of character for a genuinely open source solution. 

**Short Pixel** — Another option for use with Cache Enabler, also at a cost, is Short Pixel. Short Pixel functions much like Optimizer, described above. You can optimize 100 images a month for free; then the price jumps up. 5,000 images per month are currently $4.99 USD. 12,000 per month are $9.99. 55,000 per month are $29.99. This is more expensive than Optimizer. The difference lies in other features such as web page optimization and other non-WebP tools not covered here, but that web developers may find interesting.

**Compressing Animated GIFs and why <video> is better**

Animated GIFs continue to enjoy widespread use, despite being a very limited format. Although everything from social networks to popular media sites embed animated GIFs heavily, the format was *never* designed for video storage or animation. In fact, the[ GIF89a spec](https://www.w3.org/Graphics/GIF/spec-gif89a.txt) notes "the GIF is not intended as a platform for animation". The[ number of colors, number of frames and dimensions](http://gifbrewery.tumblr.com/post/39564982268/can-you-recommend-a-good-length-of-clip-to-keep-gifs) all impact animated GIF size. However, switching to video offers the largest savings. 


![](images/animated-gif.jpg "image_tooltip")


Delivering the same file as an MP4 video can often shave 80% or more off your file-size. This means that not only do GIFs often waste significant bandwidth, but they'll take longer to load, include fewer colors and can offer sub-part user experiences. Videos are so much better than when you upload an animated GIF to[ Twitter](http://mashable.com/2014/06/20/twitter-gifs-mp4/#fiiFE85eQZqW) or[ Imgur](https://thenextweb.com/insider/2014/10/09/imgur-begins-converting-gif-uploads-mp4-videos-new-gifv-format/), they'll silently convert it to an MP4 for you. Why are GIFs many times[ larger](https://www.quora.com/For-a-given-scene-why-does-an-animated-GIF-have-a-much-bigger-file-size-than-its-video-source-e-g-in-MP4-format)?.

Animated GIFs store each frame as a lossless GIF image - yes, lossless. The degraded quality we often experience is due to GIFs being limited to a 256-color palette. The format is often large as it doesn't consider neighbor frames for compression, unlike video codecs like H.264. An MP4 video stores each key frame as a lossy JPEG, which discards some of the original data to achieve better compression. 

**If you can switch to videos**

*   Use[ ffmpeg](https://www.ffmpeg.org/) to convert your animated GIFs (or sources) to H.264 MP4s. I use this one-liner from[ Rigor](http://rigor.com/blog/2015/12/optimizing-animated-gifs-with-html5-video):
ffmpeg -i animated.gif -movflags faststart -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" video.mp4
*   Cloudinary has an[ excellent workflow](http://cloudinary.com/blog/reduce_size_of_animated_gifs_automatically_convert_to_webm_and_mp4) for autoconverting sources (or animated GIFs) to H.264 and WEBM. I found this tip about their[ lossy mode](https://www.noupe.com/design/creating-animated-gifs-right-way-cloudinary-98607.html) useful.
*   ImageOptim API [supports this too](https://imageoptim.com/api/ungif) and also [removes dithering from GIFs](https://github.com/pornel/undither#examples) which can help video codecs compress even more.

**If you must use animated GIFs**

*   Tools like Gifsicle can strip metadata, unused palette entries and minimize what changes between frames
*   Consider a lossy GIF encoder. The [Giflossy](https://github.com/pornel/giflossy) fork of Gifsicle supports this with the `—lossy` flag and can shave ~60-65% off size. There's also a nice tool based on it called [Gifify](https://github.com/vvo/gifify).

For more information, checkout the[ Book of GIF](https://rigor.com/wp-content/uploads/2017/03/TheBookofGIFPDF.pdf) by Rigor. 


### SVG Optimization


#### Keeping SVGs lean means stripping out anything unnecessary. SVG files created with editors usually contain a large quantity of redundant information (metadata, comments, hidden layers and so forth). This content can often be safely removed or converted to a more minimal form without impacting the final SVG that's being rendered.

![](images/Modern-Image26.jpg "image_tooltip")


**Some SVG optimization good rules of thumb:**

*   Because SVGs are really just text assets expressed in XML, they can be minified to save space like CSS, HTML and JavaScript can.
Instead of paths, use *predefined* shapes in SVG like circles, squares, rectangles and polygons
*   If you must use paths, try to reduce your curves and paths. Simplify and combine them where you can. Illustrator's [simplify tool](http://jlwagner.net/talks/these-images/#/2/10) is adept at removing superfluous points in even complex artwork while smoothing out irregularities.
*   Avoid using groups. If you can't, try to simplify them.
*   Delete layers that are invisible.
*   Avoid any Photoshop or Illustrator effects. They can get converted to large raster images.
*   Double check for any embedded raster images that aren't SVG-friendly
*   Optimize your SVGs through[ SVGO](https://github.com/svg/svgo) - a Node-based tool for trimming.[ SVGOMG](https://jakearchibald.github.io/svgomg/) is a super handy web-based GUI for SVGO by Jake Archibald that I've also found invaluable. A handy [Sketch plugin for running SVGO](https://www.sketchapp.com/extensions/plugins/svgo-compressor/) on your SVGs when exporting them is also available.


![](images/svgo-precision.jpg "image_tooltip")


SVGO can also reduce file-size by lowering the *precision* of numbers in your <path> definitions. Each digit after a point adds a byte and this is why changing the precision (number of digits) can heavily influence file size. Be very very careful with changing precision however as it can visually impact how your shapes look.


![](images/svgo-precision.jpg "image_tooltip")


It's important to note that while SVGO does well in the previous example without over-simplifying paths and shapes, there are plenty of cases where this may not be the case. Observe how the light strip on the above rocket is distorted at a lower precision.

*Using SVGO at the command-line:*

SVGO can be installed as a [global npm CLI](https://www.npmjs.com/package/svgo) should you prefer that to a GUI:

```
npm i -g svgo
```

This can then be run against a local SVG file as follows:

```
svgo input.svg -o output.svg
```

It supports all the options you might expect, including adjusting floating point precision:

```
svgo input.svg --precision=1 -o output.svg
```

See the SVGO [readme](https://github.com/svg/svgo) for the full list of supported options.

**Don't forget to compress SVGs!**

![](images/before-after-svgo.jpg "image_tooltip")


Also, don't forget to [Gzip your SVG assets](https://calendar.perfplanet.com/2014/tips-for-optimising-svg-delivery-for-the-web/) or serve them using Brotli. As they're text based, they'll compress really well (~50% of the original sources).

When Google shipped a new logo, we announced that the smallest version of it was only 305 bytes in size.

![](images/Modern-Image30.jpg "image_tooltip")


There are [lots of advanced SVG tricks](https://www.clicktorelease.com/blog/svg-google-logo-in-305-bytes/) you can use to trim this down even further (all the way to 146 bytes)! Suffice to say, whether it's through tools or manual clean-up, there's probably a *little* more you can shave off your SVGs.

**Further reading**

Sara Soueidan's[ tips for optimising SVG delivery for the web](https://calendar.perfplanet.com/2014/tips-for-optimising-svg-delivery-for-the-web/) and Chris Coyier's [Practical SVG book](https://abookapart.com/products/practical-svg) are excellent. I've also found Andreas Larsen's optimizing SVG posts enlightening ([part 1](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-1-67e8f2d4035),[ part 2](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-2-6711cc15df46)).[ Preparing and exporting SVG icons in Sketch](https://medium.com/sketch-app-sources/preparing-and-exporting-svg-icons-in-sketch-1a3d65b239bb) was also a great read.


### Avoid recompressing images with lossy codecs

Recompressing images has consequences. Let's say you take a JPEG that's already been compressed with a quality of 60. If you recompress this image with lossy encoding, it will look worse. Each additional round of compression is going to introduce generational loss - information will be lost and compression artifacts will start to build up. Even if you're re-compressing at a high quality setting.

To avoid this trap, **set the lowest good quality you're willing to accept in the first place**. You then avoid this trap because any file-size reductions from quality reduction alone will look bad.


![](images/generational-loss.jpg "image_tooltip")


Re-encoding a lossy file will almost always give you a smaller file, but this doesn't mean you're getting as much quality out of it as you may think. Above, from this[ excellent video](https://www.youtube.com/watch?v=w7vXJbLhTyI) and[ accompanying article](http://cloudinary.com/blog/why_jpeg_is_like_a_photocopier) by Jon Sneyers, we can see the generational loss impact of recompression using several formats. This is a problem you may have run into if saving (already compressed) images from social networks and re-uploading them (causing recompression). Quality loss will build up.

MozJPEG (perhaps accidentally) has a better resistance to recompression degradation thanks to trellis quantization. Instead of compressing all DCT values as they are exactly, it can check close values within a +1/-1 range to see if similar values compress to fewer bits. Lossy FLIF has a hack similar to lossy PNG in that prior to (re)compression, it can look at the data and decide what to throw away. Recompressed PNGs have "holes" it can detect to avoid changing data further.

**When editing your source files, store them in a lossless format like PNG or TIFF so you preserve as much quality as you can.** Your build tools or image compression service than then handle outputting the compressed version you serve to users with minimal loss in quality. 


### Reduce unnecessary image resizes

We've all shipped large, higher resolution images than needed to our users before. This has a cost to it. Decoding and resizing images are expensive operations for a browser on average mobile hardware. If sending down large images and rescaling using CSS or width/height attributes, you're likely to see this happen and it can impact performance. 

Sending down images that a browser can render without needing to resize at all is ideal. So, serve the smallest images for your target screen sizes and resolutions, taking advantage of [`srcset` and `sizes`](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images). 


![](images/image-decoding.jpg "image_tooltip")


 When building their new mobile web experience, Twitter improved performance by ensuring they served appropriately sized images to their users. This took decode time for many images in the Twitter timeline from ~400ms all the way down to ~19!


![](images/responsive-art-direction.jpg "image_tooltip")


Although shipping the right resolution to users is important, some sites also need to think about this in terms of **[art direction](http://usecases.responsiveimages.org/#art-direction)**. If a user is on a smaller screen, you may want to zoom in and display the subject to make best use of available space. Although art direction is outside the scope of this write-up, services like[ Cloudinary](http://cloudinary.com/blog/automatically_art_directed_responsive_images%20) provide tools to try automating this as much as possible. 


### Lazy Load non-critical Images

Lazy loading is a web performance pattern that delays the loading of images in the browser until the user needs to see it. One example is as you scroll, images load asynchronously on demand. This can further compliment the byte-savings you see from having an image compression strategy.


![](images/scrolling-viewport.jpg "image_tooltip")


Images that must appear "above the fold," or when the web page first appears are loaded straight away. The images which follow "below the fold," however, are not yet visible to the user. They do not have to be immediately loaded into the browser. They can be loaded later — or lazy loaded — only if and when the user scrolls down and it becomes necessary to show them. 

Lazy loading is not yet natively supported in the browser itself (although there have been [discussions](https://discourse.wicg.io/t/a-standard-way-to-lazy-load-images/1153/10) about it in the past). Instead, we use JavaScript to add this capability.

**Why is Lazy Loading Useful?**

This "lazy" way of loading images only if and when necessary has many benefits:

* The experience can consume less data. As you aren't assuming the user will need every image fetched ahead of time you're only loading the minimal number of resources. This is always a good thing, especially on mobile with more restrictive data plans.

* Less workload for the user's browser which can save on battery life. 

Decreasing your overall page load time on an image heavy website from several seconds to almost nothing is also a tremendous boost to your user experience. In fact, it could be the difference between a user staying around to enjoy your site and just another bounce statistic.

**But like all tools, with great power comes great responsibility.**

**Avoid lazy-loading images above the fold.**

Use it for long-lists of images (e.g products) or avatars in lists of users. Don't use it for the main page hero image. Lazy-loading images above the fold can make loading visibly slower, both technically and for human perception. It can kill the browser's preloader, progressive loading and the JavaScript can create extra work for the browser.

**Who Uses Lazy Loading?**

For examples of lazy loading, look at most any major site that hosts a lot of images. Some notable sites are [Medium](https://medium.com/) and [Pinterest](https://www.pinterest.com/).


![](images/Modern-Image35.jpg "image_tooltip")


A number of sites (such as Medium) display a small, Gaussian-blurred inline preview (a few 100 bytes) that transitions (lazy-loads) to a full-quality image once it's been fetched. José M. Pérez has written about how to implement the Medium effect using [CSS filters](https://jmperezperez.com/medium-image-progressive-loading-placeholder/) and experimented with [different image formats](https://jmperezperez.com/webp-placeholder-images/) to support such placeholders. Facebook also did a write-up on their famous 200-byte approach to such placeholders for their [cover photos](https://code.facebook.com/posts/991252547593574/the-technology-behind-preview-photos/) that is worth a read. If you're a Webpack user, [LQIP loader](https://lqip-loader.firebaseapp.com/) can help automate some of this work away.

In fact, you can search for your favorite source of high-res photos and then scroll down the page. In almost all cases you'll experience how the website loads only a few full-resolution images at a time, with the rest being placeholder colors or images. As you continue to scroll, the placeholder images are replaced with full-resolution images. This is lazy loading in action.

**How Can I Apply Lazy Loading to My Pages?**

There are a number of techniques and plugins available for lazy loading. I recommend [lazysizes](https://github.com/aFarkas/lazysizes) by Alexander Farkas because of its decent performance, features, its optional integration with [Intersection Observer](https://developers.google.com/web/updates/2016/04/intersectionobserver), and support for plugins. 

**What Can I Do with Lazysizes?**

Lazysizes is a JavaScript library. It requires no configuration. Download the minified js file and include it in your webpage.


Here is some example code taken from the README file:

Add the class "lazyload" to your images/iframes in conjunction with a data-src and/or data-srcset attribute. 

Optionally you can also add a src attribute with a low quality image:

```html
<!-- non-responsive: -->

<img data-src="image.jpg" class="lazyload" />

<!-- responsive example with automatic sizes calculation: -->

<img

    data-sizes="auto"

    data-src="image2.jpg"

    data-srcset="image1.jpg 300w,

    image2.jpg 600w,

    image3.jpg 900w" class="lazyload" />

<!-- iframe example -->

<iframe frameborder="0"

    class="lazyload"

    allowfullscreen=""

    data-src="//www.youtube.com/embed/ZfV-aYdU4uE">
</iframe>
```

**Lazysizes features include:**

* Automatically detects visibility changes on current and future lazyload elements

* Includes standard responsive image support (picture and srcset)

* Adds automatic sizes calculation and alias names for media queries feature

* Can be used with hundreds of images/iframes on CSS and JS-heavy pages or web apps

* Extendable: Supports plugins

* Lightweight but mature solution

* SEO improved: Does not hide images/assets from crawlers

**More Lazy Loading Options**

Clearly, lazysizes is not your only option. Here are more lazy loading libraries:



*   [Lazy Load XT](http://ressio.github.io/lazy-load-xt/)
*   [BLazy.js](https://github.com/dinbror/blazy) (or [Be]Lazy)
*   [Unveil](http://luis-almeida.github.io/unveil/) 

**What's the catch with Lazy Loading?**



*   Screen readers, some search bots and any users with JavaScript disabled will not be able to view images lazy loaded with JavaScript. This is however something that we can work around with a `<noscript>` fallback.
*   Scroll listeners, such as used for determining when to load a lazy-loaded image, can have an adverse impact on browser scrolling performance. They can cause the browser to redraw many times, slowing the process to a crawl - however, smart lazy loading libraries will use throttling to mitigate this. One possible solution is Intersection Observer, which is supported by lazysizes. 

Lazy loading images is a widespread pattern for reducing bandwidth, decreasing costs, and improving user experience. Evaluate whether it makes sense for your experience.

Further reading:

https://jmperezperez.com/lazy-loading-images/

https://jmperezperez.com/medium-image-progressive-loading-placeholder/ 


### Does an Image Processing CDN Make Sense for You?

*The time you'll spend reading the blog posts to setup your own image processing pipeline and tweaking your config is often >> the fee for a service. With [Cloudinary](http://cloudinary.com/) offering a free service, [Imgix](https://www.imgix.com/) a free trial and [Thumbor](https://github.com/thumbor/thumbor) existing as an OSS alternative, there are plenty of options available to you for automation.*

To achieve optimal page load times, you need to optimize your image loading. This optimization calls for a responsive image strategy and can benefit from on-server image compression, auto-picking the best format and responsive resizing. What matters is that you deliver the correctly sized image to the proper device in the proper resolution as fast as possible. Doing this is not as easy as one might think.

**Using Your Server vs. a CDN**

Because of the complexity and changing nature of image manipulation, we're going to offer a quote from someone with experience in the field, then proceed with a suggestion.

"If your product is not image manipulation, then don't do this yourself. Services like Cloudinary [or imgix, Ed.] do this much more efficiently and much better than you will, so use them. And if you're worried about the cost, think about how much it'll cost you in development and upkeep, as well as hosting, storage, and delivery costs." — [Chris Gmyr](https://medium.com/@cmgmyr/moving-from-self-hosted-image-service-to-cloudinary-bd7370317a0d) 

 

For the moment, we are going to agree and suggest that you consider using a CDN for your image processing needs. Two CDNs will be examined to see how they compare relative to the list of tasks we raised earlier. 

**Cloudinary and imgix**

[Cloudinary](http://cloudinary.com/) and [imgix](https://www.imgix.com/) are two established image processing CDNs. They are the choice of hundreds of thousands of developers and companies worldwide, including Netflix and Red Bull. Let's look at them in more detail.

**What are the Basics?**

Unless you are the owner of a network of servers like they are, their first huge advantage over rolling your own solution is that they use a distributed global network system to bring a copy of your images closer to your users. It's also far easier for a CDN to "future proof" your image loading strategy as trends change - doing this on your own requires maintenance, tracking browser support for emerging formats & following the image compression community.

Second, each service has a tiered pricing plan, with Cloudinary offering a [free level](http://cloudinary.com/pricing) and imgix pricing their standard level inexpensively, relative to their high-volume premium plan. Imgix offers a free [trial](https://www.imgix.com/pricing) with a credit towards services, so it almost amounts to the same thing as a free level. 

Third, API access is provided by both services. Developers can access the CDN programmatically and automate their processing. Client libraries, framework plugins, and API documentation are also available, with some features restricted to higher paid levels. 

**Let's Get to the Image Processing**

For now, let's limit our discussion to static images. Both Cloudinary and Imgix offer a range of image manipulation methods, and both support primary functions such as compression, resizing, cropping and thumbnail creation in their standard and free plans.

![](images/Modern-Image36.jpg "image_tooltip")
 

By default Cloudinary encodes [non-Progressive JPEGs](http://cloudinary.com/blog/progressive_jpegs_and_green_martians). To opt-in to generating them, check the 'Progressive' option in 'More options' or pass the 'fl_progressive' flag.

Cloudinary lists [seven broad image transformation](http://cloudinary.com/documentation/image_transformations) categories, with a total of 48 subcategories within them. Imgix advertises over [100 image processing operations](https://docs.imgix.com/apis/url?_ga=2.52377449.1538976134.1501179780-2118608066.1501179780). 

**What Happens by Default?**

*   Cloudinary performs the following optimizations by default: 
*   [Encodes JPEGs using MozJPEG](https://twitter.com/etportis/status/891529495336722432) (opted against Guetzli as a default)
*   Strips all associated metadata from the transformed image file (the original image is left untouched). To override this behavior and deliver a transformed image with its metadata intact, add the keep_iptc flag.
*   Can generate WebP, GIF, JPEG, and JPEG-XR formats with automatic quality. To override the default adjustments, set the quality parameter in your transformation.
*   Runs [optimization](http://cloudinary.com/documentation/image_optimization#default_optimizations) algorithms to minimize the file size with minimal impact to visual quality when generating images in the PNG, JPEG or GIF format. 

Imgix has no default optimizations such as Cloudinary has. It does have a settable default image quality. For imgix, auto parameters help you automate your baseline optimization level across your image catalog. 

Currently, it has [four different methods](https://docs.imgix.com/apis/url/auto):

*   Compression
*   Visual enhancement
*   File format conversion
*   Redeye removal

Imgix supports the following image formats: JPEG, JPEG2000, PNG, GIF, Animated GIF, TIFF, BMP, ICNS, ICO, PDF, PCT, PSD, AI

Cloudinary supports the following image formats: JPEG, JPEG 2000, JPEG XR, PNG, GIF, Animated GIF, WebP, Animated WebP,BMPs, TIFF, ICOs, PDF, EPS, PSD, SVG, AI, DjVu, FLIF, TARGA

**What About Performance?**

CDN delivery performance is mostly about latency and speed. 

Latency always increases somewhat for completely uncached images. But once an image is cached and distributed among the network servers, the fact that a global CDN can find the shortest hop to the user, added to the byte savings of a properly-processed image, almost always mitigates latency issues when compared to poorly processed images or solitary servers trying to reach across the planet.

Both services use fast and wide CDN. This configuration reduces latency and increases download speed. Download speed affects page load time, and this is one of the most important metrics for both user experience and conversion. 

**So How Do They Compare?**

Cloudinary has [160K customers](http://cloudinary.com/customers) including Netflix, eBay and Dropbox. Imgix doesn't report how many customers it has, but it is smaller than Cloudinary. Even so, imgix's base includes heavyweight image users such as Kickstarter, Exposure, unsplash, and Eventbrite.  

There are so many uncontrolled variables in image manipulation that a head-to-head performance comparison between the two services is difficult. So much depends on how much you need to process the image — which takes a variable amount of time — and what size and resolution are required for the final output, which affects speed and download time. Cost may ultimately be the most important factor for you.

CDNs cost money. An image heavy site with a lot of traffic could cost hundreds of US dollars a month in CDN fees. There is a certain level of prerequisite knowledge and programming skill required to get the most out of these services. If you are not doing anything too fancy, you're probably not going to have any trouble. 

But if you're not comfortable working with image processing tools or APIs, then you are looking at a bit of a learning curve. In order to accommodate the CDN server locations, you will need to change some URLs in your local links. Do the right due diligence :)

**Conclusion**

If you are currently serving your own images or planning to, perhaps you should give a CDN some consideration.


### How do I choose an image format?

As Ilya Grigorik notes in his excellent [image optimization guide](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/image-optimization), the "right format" for an image is a combination of desired visual results and functional requirements. Are you working with Raster or Vector images?

![](images/Modern-Image37.jpg "image_tooltip")


[Raster graphics](https://en.wikipedia.org/wiki/Raster_graphics) represent images by encoding the values of each pixel within a rectangular grid of pixels. They are not resolution or zoom independent. WebP or widely supported formats like JPEG or PNG handle these graphics well where photorealism is a necessity. Guetzli, MozJPEG and other ideas we've discussed apply well to raster graphics.

[Vector graphics](https://en.wikipedia.org/wiki/Vector_graphics) use points, lines and polygons to represent images and formats using simple geometric shapes (e.g logos) offering a high-resolution and zoom like SVG handle this use case better. 

![](images/Modern-Image38.jpg "image_tooltip")


The logical flow for choosing the right format can be fraught with peril. Jeremy Wagner covers some of the considerations you might want to think about [here](http://jlwagner.net/talks/these-images/#/2/2).


### Caching image assets

Resources can specify a caching policy using [HTTP cache headers](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#cache-control). Specifically, `Cache-Control` can define who can cache responses and for how long

Most of the images you deliver to users are static assets that will[ not change](http://kean.github.io/post/image-caching) in the future. The best caching strategy for such assets is aggressive caching. 

When setting your HTTP caching headers, set Cache-Control with a max-age of a year (e.g `Cache-Control:public; max-age=31536000`). This type of aggressive caching works well for most types of images, especially those that are long-lived like avatars and image headers.

<aside class="note"><b>Note:</b> If you're serving images using PHP, it can destroy caching due to the default [session_cache_limiter](http://php.net/manual/en/function.session-cache-limiter.php) setting. This can be a disaster for image caching and you may want to [work around](https://stackoverflow.com/a/3905468) this by setting session_cache_limiter('public') which will set public, max-age=. Disabling and setting custom cache-control headers is also fine.</aside>


### Closing recommendations

Ultimately, choosing an image optimization strategy will come down to the types of images you're serving down to your users and what you decide is a reasonable set of evaluation criteria. It might be using SSIM or Butteraugli or, if it's a small enough set of images, going off of human perception for what makes the most sense. 

**Here are my closing recommendations:**

If you **can't** invest in conditionally serving formats based on browser support:



*   Guetzli + MozJPEG's jpegtran is a good format for JPEG quality > 90. 
    *   For the web q=90 is wastefully high. You can get away with q=80, and on 2x displays even with q=50. Since Guetzli doesn't go that low, for the web you can MozJPEG. 
    *   Kornel recently improved mozjpeg's cjpeg command to add tiny sRGB profile to help Chrome display natural color on wide-gamut displays
*   PNG pngquant + advpng has a pretty good speed/compression ratio

If you **can** conditionally serve (using <picture>, the Accept header or Picturefill):



*   Serve WebP down to browsers that support it (e.g Chrome, Opera)
    *   Create WebP images from original 100% quality images. Otherwise you'll be giving browsers that do support it worse-looking images with JPEG distortions *and* WebP distortions! If you compress uncompressed source images using WebP it'll have the less visible WebP distortions and can compress better too.
    *   The default settings the WebP team use of `-m 4 -q 75` are usually good for most cases where they optimize for speed/ratio.
    *   WebP also has a special mode for lossless (`-m 6 -q 100`) which can reduce a file to its smallest size by exploring all parameter combinations. It's an order of magnitude slower but is worth it for static assets.
*   As a fallback, serve Guetzli/MozJPEG compressed sources to other browsers

Happy compressing!

<aside class="note"><b>Note:</b> For more practical guidance on how to optimize images, I heavily recommend [Web Performance in Action](https://www.manning.com/books/web-performance-in-action) by Jeremy Wagner. It's a fantastic book. [High Performance Images](http://shop.oreilly.com/product/0636920039730.do) is also filled with excellent, nuanced advice on this topic.</aside>

With thanks to our tech reviewers: Kornel Lesinski, Jeremy Wagner, Tim Kadlec, Nolan O'Brien and Kristofer Baxter.

</body>
</html>