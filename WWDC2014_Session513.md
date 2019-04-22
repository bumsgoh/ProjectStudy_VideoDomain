AVKit -> AVFoundation -> Video Toolbox -> Core Media -> Core Video



AVFoundation

- 압출을 풀어 바로 디스플레이 가능

- 파일로 바로 압축가능

  

Video Toolbox

- CVPixelBuffer로 압출 풀기가능
- CMSampleBuffer로 압축가능



키워드 

- CVPixelBuffer
  - 이미지 데이터 블락을 포함한다. CVPixelBuffer가 이 (Uncompressed Raster Image Buffer) 를 래핑하고 있고 어떻게 접근하여야 하는지 알려준다.
- CVPixelBufferPool
  - CVPixelBuffer를 재사용할 수 있게 도와주는 풀이다.
- pixelBufferAttributes
  - CFDictionary : width/height , 픽셀 형식(32BGRA, YCbCr420), 호환성(OpenGL ES, CoreAnimation) 등의 정보를 포함한다.
- CMTime
  - 64 bit 분자 (Time Value) , 32bit 분모(Time Scale)
- CMVideoFormatDescription
  - 비디오 데이터에 대한 설명이다. Width/Height, Format Type(KCMPixelFormat_32BGRA, KCMVideoCodecType_H264), Extensions(Pixel Aspect Ratio, Color Space)
- CMBlockBuffer
  - 코어 미디어에서 임의의 데이터를 래핑하는 용도
- CMSampleBuffer
  - 압축된 비디오 프레임 일 경우 그리고 압축되지 않은 Raster Image
    (픽셀 이미지)의 경우 두 가지로 나뉜다. 압축된 비디오 프레임은 CMTIme, CMVideoFormatDesc, CMBlockBuffer를 내부에 포함하지만 래스터 이미지는 CMBlockBuffer대신 CVPixelBuffer를 포함한다.
- CMClock
  - 코어 미디어가 시간정보를 래핑한 것이다. 시간이 계속 증가하는 특징을 가진다.
- CMTimebase
  - CMClock을 기준으로 타임 매핑과 rate 제어를 할 수 있다.



네트워크 스트리밍하는 케이스

네트워크 -> 압축된 비디오 샘플(H264) -> AVSampleBufferDisplayLayer -> APP



AVSampleBufferDisplayLayer

- CMSampleBuffers(H.264)를 받아서 내부적으로 비디오 디코더를 가지고 있어 데이터를 CVPixelBuffers로 디코딩할 수 있다

하지만 기본적으로 네트워킹을 통해서 가져오는 데이터는 로우하다. 이를 CMSampleBuffers로 바꾸려면 어떠한 처리가 필요하다.



H.264 Syntax 에서는 Elementary Stream의 경우 MPEG-4의 경우가 있다. 

프레임워크가 기본적으로 MPEG-4에 대응할 수 있으나 Elemantary에는 대응할 수 없다.

H.264 스트림은 NALU 시퀀스로 이루어져 있고 이에 대한 설명은 NALU 문서를 참고하면 된다.



Elementary Stream 에서는 Parameter Sets가 스트림 내부에 포함되어 있으며 이와 달리 MPEG-4에서는 Format Description안에 해당  Parameter Sets가 모아져 있다.(CMVideoFormatDescription) 

그러므로 Elementary Stream 경우 데이터를 MPEG-4로 재 포장 해줘야한다. (Iframe/BFrame/PFrame?)

위 동작을 해주는 메서드가 바로 CMVideoFormatDescriptionCreateFromH264ParameterSets이다
