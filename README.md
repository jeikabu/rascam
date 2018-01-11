http://www.jvcref.com/files/PI/documentation/html/index.html
http://vojtech.kral.hk/en/rust-ffi-wrapping-c-api-in-rust-struct/
http://siciarz.net/ffi-rust-writing-bindings-libcpuid/

TODO:
* decide on API
* see what we can do about using Rust's memory allocator instead of `vcos`? malloc
* match up capture method and complete status from callback function. this is currently very unsafe!
* try to reduce unsafe rust to minimum
* anything else unsafe?

# API ideas

## New SimpleCamera

Do we want to force users to use futures? Should we provide a callback-style API? Should `capture_still()` just block?

Note that these C types should actually be wrappers, not raw types or they are likely to cause memory management fun.

There should be settings for brightness, exposure, burst mode, image format etc etc.

Should we provide a preview API?

### `SimpleCamera::new() -> SimpleCamera`

### `set_camera_num(u8) -> Result<MMAL_PARAMETER_CAMERA_INFO_CAMERA_T, MMAL_STATUS_T>`

### `set_camera_info(MMAL_PARAMETER_CAMERA_INFO_CAMERA_T) -> Result<(), MMAL_STATUS_T>`

This is a companion for the above. Useful if the user already has a `MMAL_PARAMETER_CAMERA_INFO_CAMERA_T`. Does it provide enough value?

### `activate() -> Result<(), MMAL_STATUS_T>`

Start the camera. Useful for metering etc.

### `capture_still() -> Future<[u8], MMAL_STATUS_T>`

Take a still picture.

What about burst mode? `capture_burst() -> Stream<[u8], MMAL_STATUS_T>`?

### `record_video() -> Stream<[u8], MMAL_STATUS_T>`

Record a video and return a stream of frames.

### `stop_video()`

Should this return a `Result<(), ?>` so that we can error if not already recording a video?

## Old SimpleCamera

Is there much need for `Result` when we have `MMAL_SUCCESS`? Not really but `Result` is Rust-like and `MMAL_SUCCESS` is C-like.

How should we represent errors? Is just `MMAL_STATUS_T` ok?
Is this informative? Should we have a `CameraError` type with a code property? Is this better? Is it enough?

What happens when there are multiple libmmal calls inside a method? Is it clear which the error comes from or what it means?

### `SimpleCamera::new() -> Result<SimpleCamera, MMAL_STATUS_T>`

Should this actually create camera objects? (it does now)  
Should this take the camera number?  
Should this take a `MMAL_PARAMETER_CAMERA_INFO_CAMERA_T`?

### `set_camera_num(u8) -> Result<(), MMAL_STATUS_T>`

If constructor doesn't take a camera number or camera info, we
should get one here.

Users or SimpleCamera shouldn't care about any of the following APIs except `take()`.

### `create_encoder() -> Result<(), MMAL_STATUS_T>`

### `enable_control_port() -> Result<(), MMAL_STATUS_T>`

### `set_camera_params(MMAL_PARAMETER_CAMERA_INFO_CAMERA_T) -> Result<(), MMAL_STATUS_T>`

Users shouldn't have to pass this in.

### `set_camera_format(MMAL_PARAMETER_CAMERA_INFO_CAMERA_T) -> Result<(), MMAL_STATUS_T>`

Users shouldn't have to pass this in and certainly not twice.

### `enable() -> Result<(), MMAL_STATUS_T>`

### `create_pool() -> Result<(), MMAL_STATUS_T>`

### `create_preview() -> Result<(), MMAL_STATUS_T>`

### `enable_preview() -> Result<(), MMAL_STATUS_T>`

### `connect_ports() -> Result<(), MMAL_STATUS_T>`

### `enable_still_port() -> Result<(), MMAL_STATUS_T>`

### `take() -> Result<(), MMAL_STATUS_T>`

Rename to capture?

## SeriousCamera (or just Camera?)

## CameraInfo

`CameraInfo::info() -> Result<CameraInfo, MMAL_STATUS_T>`

# Debugging

```
$ convert --version
Version: ImageMagick 6.9.9-27 Q16 x86_64 2017-12-23 http://www.imagemagick.org
Copyright: © 1999-2018 ImageMagick Studio LLC
License: http://www.imagemagick.org/script/license.php
Features: Cipher DPC Modules OpenMP
Delegates (built-in): bzlib cairo djvu fftw fontconfig freetype gslib jbig jng jp2 jpeg lcms ltdl lzma openexr pangocairo png ps rsvg tiff webp wmf x xml zlib
```

To test rgb output:
```
convert -size 96x96 -depth 8 -colorspace RGB rgb:test.rgb out.png
```
