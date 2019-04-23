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



## NALU 스타트 코드

NALU는 자신의 크기를 포함하고 있지 않기 때문에 스트림을 생성하기 위해 그저 NALU들을 이어 붙이는 것은 작동하지 않을 것이다. 왜냐하면 어디서 NALU가 끝나고 시작하는지 전혀 모르기 때문이다. Annex B 명세는 NALU 앞에 "스타트 코드"를 요구함으로서 이 문제를 해결했다. 스타트 코드는 2 또는 3 개의 0x00 바이트 뒤에 0x01 바이트가 나오는 방식이다. 즉, 00 00 01 이나 00 00 00 01 이 그 예가 될 수 있다.

4바이트 방식은 시리얼 연결을 통한 통신에 유용하다 왜냐하면 다른 방식으로는 31개의 0비트를 찾고 그 다음에 1개의 0비트를 찾아 스트림을 바이트 정렬해야 하기 때문이다. 만약 다음 비트가 0 이라면 (모든 NALU는 0으로 시작한다) 그것은 다음 NALU의 시작이다. 4바이트 방식은 보통 SPS PPS AUD 그리고 IDR 같은 랜덤 액세스 포인트에 시그널을 줄때만 사용되며 3바이트 방식은 공간을 절약하기 위해 사용된다.

 

## Emulation Prevention 바이트

스타트 코드는 4바이트 시퀀스 00 00 00, 00 00 01, 00 00 02, 00 00 03 가  non-RBSP NALU내에서 적용되지 않으므로서 동작할 수 있다. 그래서 NALU를 만들때 스타트 코드와 헷갈리지 않기위해서 이러한 숫자에 신경을 써주어야 한다. 이것은 Emulation Prevention 바이트를 추가함으로서 가능하다. 0x03이 바로 그 수이다.(00 00 01 은 00 00 301이 된다.)

디코딩을 할 때 Emulation Prevention 바이트를 찾아내어 무시해주는 것이 중요하다. 왜냐하면 Emulation Prevention 바이트들은 거의 모든 NALU에 있을 수 있고 문서상에서는 그것을이 이미 처리가 되었다고 가정하는 것이 훨씬 편하기 때문이다. Emulation Prevention 바이트가 없는 방식을  Raw Byte Sequence Payload (RBSP)라고 부른다.

## 예시

Let's look at a complete example.

```
0x0000 | 00 00 00 01 67 64 00 0A AC 72 84 44 26 84 00 00
0x0010 | 03 00 04 00 00 03 00 CA 3C 48 96 11 80 00 00 00
0x0020 | 01 68 E8 43 8F 13 21 30 00 00 01 65 88 81 00 05
0x0030 | 4E 7F 87 DF 61 A5 8B 95 EE A4 E9 38 B7 6A 30 6A
0x0040 | 71 B9 55 60 0B 76 2E B5 0E E4 80 59 27 B8 67 A9
0x0050 | 63 37 5E 82 20 55 FB E4 6A E9 37 35 72 E2 22 91
0x0060 | 9E 4D FF 60 86 CE 7E 42 B7 95 CE 2A E1 26 BE 87
0x0070 | 73 84 26 BA 16 36 F4 E6 9F 17 DA D8 64 75 54 B1
0x0080 | F3 45 0C 0B 3C 74 B3 9D BC EB 53 73 87 C3 0E 62
0x0090 | 47 48 62 CA 59 EB 86 3F 3A FA 86 B5 BF A8 6D 06
0x00A0 | 16 50 82 C4 CE 62 9E 4E E6 4C C7 30 3E DE A1 0B
0x00B0 | D8 83 0B B6 B8 28 BC A9 EB 77 43 FC 7A 17 94 85
0x00C0 | 21 CA 37 6B 30 95 B5 46 77 30 60 B7 12 D6 8C C5
0x00D0 | 54 85 29 D8 69 A9 6F 12 4E 71 DF E3 E2 B1 6B 6B
0x00E0 | BF 9F FB 2E 57 30 A9 69 76 C4 46 A2 DF FA 91 D9
0x00F0 | 50 74 55 1D 49 04 5A 1C D6 86 68 7C B6 61 48 6C
0x0100 | 96 E6 12 4C 27 AD BA C7 51 99 8E D0 F0 ED 8E F6
0x0110 | 65 79 79 A6 12 A1 95 DB C8 AE E3 B6 35 E6 8D BC
0x0120 | 48 A3 7F AF 4A 28 8A 53 E2 7E 68 08 9F 67 77 98
0x0130 | 52 DB 50 84 D6 5E 25 E1 4A 99 58 34 C7 11 D6 43
0x0140 | FF C4 FD 9A 44 16 D1 B2 FB 02 DB A1 89 69 34 C2
0x0150 | 32 55 98 F9 9B B2 31 3F 49 59 0C 06 8C DB A5 B2
0x0160 | 9D 7E 12 2F D0 87 94 44 E4 0A 76 EF 99 2D 91 18
0x0170 | 39 50 3B 29 3B F5 2C 97 73 48 91 83 B0 A6 F3 4B
0x0180 | 70 2F 1C 8F 3B 78 23 C6 AA 86 46 43 1D D7 2A 23
0x0190 | 5E 2C D9 48 0A F5 F5 2C D1 FB 3F F0 4B 78 37 E9
0x01A0 | 45 DD 72 CF 80 35 C3 95 07 F3 D9 06 E5 4A 58 76
0x01B0 | 03 6C 81 20 62 45 65 44 73 BC FE C1 9F 31 E5 DB
0x01C0 | 89 5C 6B 79 D8 68 90 D7 26 A8 A1 88 86 81 DC 9A
0x01D0 | 4F 40 A5 23 C7 DE BE 6F 76 AB 79 16 51 21 67 83
0x01E0 | 2E F3 D6 27 1A 42 C2 94 D1 5D 6C DB 4A 7A E2 CB
0x01F0 | 0B B0 68 0B BE 19 59 00 50 FC C0 BD 9D F5 F5 F8
0x0200 | A8 17 19 D6 B3 E9 74 BA 50 E5 2C 45 7B F9 93 EA
0x0210 | 5A F9 A9 30 B1 6F 5B 36 24 1E 8D 55 57 F4 CC 67
0x0220 | B2 65 6A A9 36 26 D0 06 B8 E2 E3 73 8B D1 C0 1C
0x0230 | 52 15 CA B5 AC 60 3E 36 42 F1 2C BD 99 77 AB A8
0x0240 | A9 A4 8E 9C 8B 84 DE 73 F0 91 29 97 AE DB AF D6
0x0250 | F8 5E 9B 86 B3 B3 03 B3 AC 75 6F A6 11 69 2F 3D
0x0260 | 3A CE FA 53 86 60 95 6C BB C5 4E F3
```

위 표는 NALU 3개를 포함하는 완전한 AU이다. 보이듯이 우리는 스타트 코드 바로 다음에 67로 표현되는 SPS를 가지고 있고 그 안에 2개의 Emulation Prevention 바이트가 보일것이다. 이 바이트들이 없으면 00 00 00 같은 적합하지 않은 시퀀스가 생기게 된다. 다음으로 PPS가 68로 시작하는 것을 볼 수 있다. 그리고 마지막 스타트 코드 뒤로 IDR 슬라이스가 따라온다. 이것이 바로 완전한 H.264 스트림인 것이다. 만약 이 값들을 16진법으로 바꾸고 확장자를 .264로 한다면 아래의 이미지를 확인할 수 있을 것이다.



![Lena](https://i.stack.imgur.com/Szfku.png)





Annex B는 보통 TS같이 방송되거나 DVD 같은 라이브와 스트리밍 형식으로 사용된다. 이 형식에서 SPS PPS 가 주기적으로 반복되는 것이 일반적이며 보통은 모든 IDR앞에 오게된다. 그라므로 디코더를 위한 랜덤 액세스 포인트가 생성될 수 있는 것이다. 이미 진행되고 있는 스트림에 통합될 수 있는 기능을 이로써 부여할 수 있게된다.

