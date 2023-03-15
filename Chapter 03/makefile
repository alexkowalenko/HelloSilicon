#TOOLPATH = $(shell dirname $(shell xcodebuild -find clang))
TOOLPATH = /opt/homebrew/opt/llvm/bin

HelloWorld: HelloWorld.o
	$(TOOLPATH)/ld64.lld -platform_version macos 13 0 -o HelloWorld HelloWorld.o -lSystem -syslibroot `xcrun -sdk macosx --show-sdk-path` -e _start -arch arm64

HelloWorld.o: HelloWorld.s
	$(TOOLPATH)/clang -o HelloWorld.o -g -c HelloWorld.s

clean: 
	rm HelloWorld HelloWorld.o
