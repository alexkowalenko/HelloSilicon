AS=clang -x assembler-with-cpp -static -nostdlib -c
LD=ld64.lld -platform_version macos 13 0 -arch arm64 

HelloWorld: HelloWorld.o makefile
	$(LD) -o HelloWorld HelloWorld.o -e _start

HelloWorld.o: HelloWorld.S makefile
	$(AS) -o HelloWorld.o HelloWorld.S

clean:
	rm HelloWorld.o HelloWorld
