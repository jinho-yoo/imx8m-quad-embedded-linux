## 🎙️ Scenario 1: 빌드 결과물의 위치와 종류를 묻는 질문  
Interviewer: "Once the Bitbake process is finished, where do you find the output and what are the key artifacts?"  

You: "After a successful build, I head over to the tmp/deploy/images directory. The most critical file is the .wic.bz2 image. For instance, if I'm building a light system, I'll get core-image-minimal-imx8mqevk.wic.bz2, which is a compressed, bootable disk image.  

If the project requires full hardware acceleration, I'd go with imx-image-multimedia-imx8mqevk.wic.bz2. This one is much larger because it includes the VPU and GPU drivers along with the GStreamer stack needed for high-performance media playback on the i.MX8M."  
  
## 🎙️ Scenario 2: 부가 파일(.manifest, .json)의 용도를 묻는 질문  
Interviewer: "I see other files like .manifest or .json in the deploy folder. Do you actually use those?"  

You: "Absolutely. They are vital for traceability and testing. I use the .manifest file to audit exactly which packages and versions are baked into the image—it's essentially the 'Bill of Materials' for my software.  

The .testdata.json is also great for automated testing environments. It contains metadata about the build configuration, which helps in reproducing the exact same environment if we encounter a bug later in the QA stage."  
  
## 🎙️ Scenario 3: 실제 보드에 굽는 과정(Flashing)을 묻는 질문  
Interviewer: "How do you typically flash these images to your EVK (Evaluation Kit)?"  

You: "I usually use the WIC images because they contain the entire partition table, including the bootloader and rootfs. For the i.MX8M, I'd take that .wic.bz2 file and flash it to an SD card using a tool like bmaptool. It’s much more efficient than dd because it understands the sparse blocks in the image, saving a lot of time during the development cycle."  
  
## 💡 면접관을 사로잡는 '프로의 팁' (Key Vocabulary)  
면접 중에 아래 단어들을 섞어서 사용하면 훨씬 전문적으로 보입니다.  
Artifacts: 빌드 결과물을 뜻하는 전문 용어입니다.  
Traceability: 나중에 문제가 생겼을 때 추적 가능한 능력을 말하며, .manifest 파일의 존재 이유입니다.  
Headless: core-image-minimal처럼 화면(UI)이 없는 시스템을 말할 때 씁니다.  
Hardware Acceleration: GPU/VPU 가속을 뜻하며, imx-image-multimedia의 존재 이유입니다.  
  
