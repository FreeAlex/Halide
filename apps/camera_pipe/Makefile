GXX ?= g++-4.7

PNGFLAGS=$(shell libpng-config --ldflags) $(shell libpng-config --cflags)
ifeq ($(HL_TARGET),ptx)
  SCHEDULE=100
  CUDA_LFLAGS=-L/usr/local/cuda/lib -lcuda
else
  SCHEDULE=0
endif

camera_pipe: ../../cpp_bindings/ ../../src/ camera_pipe.cpp
	$(MAKE) -C ../../cpp_bindings libHalide.a
	$(GXX) -g camera_pipe.cpp -I ../../cpp_bindings/ ../../cpp_bindings/libHalide.a -o camera_pipe -ldl -lpthread

curved.bc: camera_pipe
	./camera_pipe 8 0 # 8-bit output,

curved.o: curved.bc
	cat curved.bc | ../../llvm/Release+Asserts/bin/opt -O3 | ../../llvm/Release+Asserts/bin/llc -O3 -filetype=obj > curved.o

fcam/Demosaic.o: fcam/Demosaic.cpp fcam/Demosaic.h
	$(GXX) -c -I../../support -Wall -fopenmp -O3 $< -o $@

fcam/Demosaic_ARM.o: fcam/Demosaic_ARM.cpp fcam/Demosaic_ARM.h
	$(GXX) -c -I../../support -Wall -fopenmp -O3 $< -o $@

process: process.cpp curved.o fcam/Demosaic.o fcam/Demosaic_ARM.o
	$(GXX) -I../../support -Wall -O3 $^ -o $@ -lpthread -ldl -fopenmp $(CUDA_LFLAGS) $(PNGFLAGS)

out.png: process raw.png
	./process raw.png 3700 2.0 50 out.png

clean:
	rm -f out.png process curved.o curved.bc camera_pipe
