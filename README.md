# dslr-timelapse
Making a timelapse with a DSLR + gphoto + darktable + ffmpeg

## 1. Taking the photos
### Equipment
- Camera (needs to be compatible with [gphoto2](http://gphoto.org/proj/libgphoto2/support.php))
- AC Adapter for camera (only required if the duration is longer than the battery life of the camera)
- PC (computer/laptop/raspberry pi)
- USB cable to connect camera to PC
- Tripod/stand

### Setup
- Install gphoto2

```
sudo apt-get install gphoto2
```
- Set-up the camera (place on tripod, plug AC adapter into camera, connect USB cable between camera and PC
- Turn on camera
- Check camera connects + is detected by gphoto2 (for gphoto2's command reference, [see here](http://gphoto.org/doc/manual/ref-gphoto2-cli.html))
```
gphoto2 --auto-detect
```
- Check commands supported by the camera
```
gphoto2 --abilities
```
- Check that you can capture + download an image
```
gphoto2 --capture-image-and-download
```
- Check configuration options of camera. Save to file, to reference for creating command string later
```
gphoto2 --list-all-config
```
### Create command string
- For consistent images, fix the f-stop, aperture, ISO, exposure, and focus (this is an example for the Canon D350, use your configuration options to create your own string
```
--set-config /main/settings/iso=400 --set-config /main/settings/shutterspeed=1/400 --set-config /main/settings/iso=400  --set-config /main/settings/exposurecompensation=0  --set-config /main/settings/focusmode=3
```
- Specify the file type that you want (on this camera, format 9 is for RAW file + small JPEG)
```
--set-config /main/settings/flashmode=0
```
- Specify time between shots (in seconds)
```
--interval=15
```
- Put together your full capture string
```
gphoto2 --set-config /main/settings/imageformat=9 --set-config /main/settings/iso=400 --set-config /main/settings/shutterspeed=1/400 --set-config /main/settings/iso=400  --set-config /main/settings/exposurecompensation=0  --set-config /main/settings/focusmode=3  --set-config /main/settings/flashmode=0 --capture-image-and-download --interval=15
```
> Note: This will capture continuously (stop with ctrl + c). You can also specify a number of frames with --frames X

### To do: Add hook to image processing

### Start the capture
- Run the capture string above, and verify that images are downloaded to the local folder

## 2. Processing the images
### Setup
- Install [darktable](https://www.darktable.org/)
```
sudo apt-get install darktable
```
- Open darktable. We need to process one image to replicate this processing for all future images
..- Apply lens correction (Check if your lens is supported by lensfun [here](http://lensfun.sourceforge.net/lenslist/)
..- Apply any other filtering/exposure adjustment/rotation/color correction/etc.
- An IMAGE_FILE_NAME.xmp file will be created in the same directory as the image file. Rename this to template.xmp - to be used later
### Process the images
- This can also be added as a hook, to be processed immediately after the photos are taken
- Loop through all image/raw files, and apply darktable preset
```
for i in $(ls *.CR2);
do
	darktable-cli $i template.xmp corrected/$i.jpg;
done
```
## 3. Create the video
### Setup
- Install [ffmpeg](https://ffmpeg.org/)
```
sudo apt-get install ffmpeg
```
- Basic timelapse
```
ffmpeg -r 60 -start_number 1234 -i _MG_%4d.CR2.jpg -c:v libx264 -pix_fmt yuv420p -vf scale=1920:-2 timelapse.mp4
```
  - `-r 60` specifies the input frame-rate (of the images). Higher = faster timelapse
  - `-start_number 1234` specifies the first sequence number of the images
  - `-i _MG%4d.CR2.jpg` specifies the input file name. `%4d` is a four digit (leading zeros) number, eg. `_MG1234.CR2.jpg`
  - `-c:v libx264` is using the H264 video enconding standard
  - `-pix_fmt yuv420p` using yuv color space, with 4:2:0 chroma subsampling
  - `-vf scale=1920:-2` scales to 1920 horizontal pixels, keeping the same aspect ratio. 
  - `timelapse.mp4` is the output file name

- Add more filters (eg. a zoom)
```
ffmpeg -r 60 -start_number 4034 -i _MG_%4d.CR2.jpg -c:v libx264 -pix_fmt yuv420p -vf "deshake=x=1506:y=1408:w=400:h=70:edge='original':filename='motion',zoompan=z='1+(1*in/700)':x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)':d=1, scale=1920:-2" timelapse-zoom-deshake.mp4
```

## 4. Result
[![Video timelapse of Olympic Stadium, Montreal](https://img.youtube.com/vi/ErL1pw--id8/0.jpg)](https://www.youtube.com/watch?v=ErL1pw--id8)
