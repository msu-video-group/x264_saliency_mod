# x264_saliency_mod
A fork of x264 video encoder supporting custom saliency maps as an additional input to improve quality of salient objects.

An example (click to see the full-size images):

| Ordinary x264 | Saliency-aware x264 |
|---------------|---------------------|
| [![Ordinary x264](http://compression.ru/video/savam/images/github/0003_015651_x264_crop.jpg)](http://compression.ru/video/savam/images/github/0003_015651_x264.png) | [![Saliency-aware x264](http://compression.ru/video/savam/images/github/0003_015651_proposed_crop.jpg)](http://compression.ru/video/savam/images/github/0003_015651_proposed.png) |

# Getting started
You could download a build [here](https://github.com/msu-video-group/x264_saliency_mod/releases). 
The archive contains an executable for Windows, if you are a Linux user, check `example.sh` scirpt in the archive to run the codec through `wine`.

Then you could compress videos taking into account their visual attention distribution as follows:

```bash
Windows:
x264.exe "source.avi" --output "source-saliency-aware.mkv" --bitrate 1000 --saliency "saliency.avi" --saliency-s0 "75%%" --saliency-bitrate "40%%"

Linux:
wine x264.exe "source.avi" --output "source-saliency-aware.mkv" --bitrate 1000 --saliency "saliency.avi" --saliency-s0 75% --saliency-bitrate 40%
```

The first three parameters are the same as in ordinary x264 encoder:
* `"source.avi"` is a path to source video;
* `--output` specifies a path to the compressed video;
* `--bitrate` specifies the average bitrate of the compressed video.

The following three parameters are the most interesting, they are responsible for saliency-aware compression:
* `--saliency` specifies a path to a video with saliency of source video, the more bright pixels the more bitrate they will get;
* `--saliency-s0` and `--saliency-bitrate` specify how much bitrate salient areas will get, in this example 25% of the most salient pixels will get 40% of the bitrate;
* you can find more information about these parameters in `x264.exe --help` -> section `Saliency-aware compression`.

# How it works
You could read explanation of the algorithm in the original article "A semiautomatic saliency model and its application
to video compression".
The paper is available [here](http://compression.ru/video/savam/pdf/A_semiautomatic_saliency_model_and_its_application_to_video_compression_ICCP_2017_0.pdf).

More details are available at [project page](http://compression.ru/video/savam/#downloads).
