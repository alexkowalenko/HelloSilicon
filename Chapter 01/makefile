AS=as
LD=ld64.lld -platform_version macos 13 0

HelloWorld: HelloWorld.o
	ld -o HelloWorld HelloWorld.o -lSystem -syslibroot `xcrun -sdk macosx --show-sdk-path` -e _start -arch arm64 

HelloWorld.o: HelloWorld.S
	as -o HelloWorld.o -c HelloWorld.S

clean:
	rm HelloWorld.o HelloWorld
