NALU : Network Abstraction Layer Units

- 패킷은 Network Abstraction Layer Units 라고 불리며 자주 NALU 로 줄여서 말한다. 각 패킷은 개별적으로 파싱되고 처리될수 있다. 각 NALU 의 첫 번째 바이트는 NALU 타입을 포함하고 있고 특히 3에서 7 비트이다. (0은 제외되고 1-2는 어떤 NALU가 다른 NALU에 의해 참조되고 있는지 나타낸다.) 

- 19개의 NALU 타입이 두 개의 카테고리로 나누어 정의되어있다. 

  - VCL(Video Coding Layer) 패킷은 실제 시각정보를 포함한다
  - Non - VCL 은 비디오 디코딩에 필요한 메타데이터를 포함할 수 있다

  한개의 NALU 또는 VCL NALU는 프레임을 의미하는것이 아니다. 프레임은 슬라이스되어 몇개의 NALU 로 나뉠 수 있다. 한개 또는 여러개의 슬라이스가 가상적으로 그룹화하여 AU(Access Unit)을 이루며 이것이 하나의 프레임을 포함하게 된다. 슬라이싱은 비용이 많이 발생하여 자주 사용되지 않는다. 

- NALU 테이블

```
0      Unspecified                                                 non-VCL
1      Coded slice of a non-IDR picture                               VCL
2      Coded slice data partition A                                   VCL
3      Coded slice data partition B                                   VCL
4      Coded slice data partition C                                   VCL
5      Coded slice of an IDR picture                                  VCL
6      Supplemental enhancement information (SEI)                     non-VCL
7      Sequence parameter set                                         non-VCL
8      Picture parameter set                                          non-VCL
9      Access unit delimiter                                          non-VCL
10     End of sequence                                                non-VCL
11     End of stream                                                  non-VCL
12     Filler data                                                    non-VCL
13     Sequence parameter set extension                               non-VCL
14     Prefix NAL unit                                                non-VCL
15     Subset sequence parameter set                                  non-VCL
16     Depth parameter set                                            non-VCL
17..18 Reserved                                                       non-VCL
19     Coded slice of an auxiliary coded picture without partitioning non-VCL
20     Coded slice extension                                          non-VCL
21     Coded slice extension for depth view components                non-VCL
22..23 Reserved                                                       non-VCL
24..31 Unspecified                                                    non-VCL
```




- NALU 자주 쓰이는 타입

- **Sequence Parameter Set (SPS).** 이 타입은 non-VCL NALU 로서 프로파일이나, 레벨, 레졸루션, 프레임 레이트를 조정할 때 필요한 정보를 담고 있다.
- **Picture Parameter Set (PPS).** SPS와 비슷하게 이 non-VCL 타입은 엔트로피 코딩모드, 슬라이스 그룹, 모션예측 그리고 디블록킹 필터에 관한 정보를 포함한다.
- **Instantaneous Decoder Refresh (IDR).** 이 VCL NALU 타입은 이미지가 슬라이스된 것 자체이다. 그러니까 IDR은 SPS 나 PPS를 저장한 다른 NALU를 참조하지 않고도 디코드 되고 출력될 수 있다. 
- **Access Unit Delimiter (AUD).** AUD는 선택적인 NALU로서 로우 스트림에서 프레임 한계치를 정하는데에 사용된다. TS같은 컨테이너나 프로토콜에 언급되지 않는이상 필수적인 요소는 아니며 공간절약을 위해 자주 생략된다. 하지만 모든 NALU를 파싱하지 않고도 시작점은 찾을 수 있다는 점에서 유용하다.


출처 : https://stackoverflow.com/questions/24884827/possible-locations-for-sequence-picture-parameter-sets-for-h-264-stream/24890903#24890903
