#---------------------------------------------------------------------
# Makefile for Rotor
#
# Author : Jean-Luc PONS

SRC = Base58.cpp IntGroup.cpp Main.cpp Bloom.cpp Random.cpp Sort.cpp \
      Timer.cpp Int.cpp IntMod.cpp Point.cpp SECP256K1.cpp \
      Rotor.cpp GPU/GPUGenerate.cpp hash/ripemd160.cpp \
      hash/sha256.cpp hash/sha512.cpp hash/ripemd160_sse.cpp \
      hash/sha256_sse.cpp hash/keccak160.cpp GmpUtil.cpp CmdParse.cpp

OBJDIR = obj

ifdef gpu

OBJET = $(addprefix $(OBJDIR)/, \
        Base58.o IntGroup.o Main.o Bloom.o Random.o Sort.o Timer.o Int.o \
        IntMod.o Point.o SECP256K1.o Rotor.o GPU/GPUGenerate.o \
        hash/ripemd160.o hash/sha256.o hash/sha512.o \
        hash/ripemd160_sse.o hash/sha256_sse.o hash/keccak160.o \
        GPU/GPUEngine.o GmpUtil.o CmdParse.o)

else

OBJET = $(addprefix $(OBJDIR)/, \
        Base58.o IntGroup.o Main.o Bloom.o Random.o Sort.o Timer.o Int.o \
        IntMod.o Point.o SECP256K1.o Rotor.o GPU/GPUGenerate.o \
        hash/ripemd160.o hash/sha256.o hash/sha512.o \
        hash/ripemd160_sse.o hash/sha256_sse.o hash/keccak160.o \
        GmpUtil.o CmdParse.o)

endif

CXX        = g++
CUDA       = /usr/local/cuda
CXXCUDA    = /usr/bin/g++
NVCC       = $(CUDA)/bin/nvcc
# nvcc requires joint notation w/o dot, i.e. "5.2" -> "52"
ccap       = $(shell echo $(CCAP) | tr -d '.')
ifdef pybind
PYBIND_FLAGS = -shared -fPIC $(shell python3 -m pybind11 --includes)
endif

ifdef gpu
ifdef debug
CXXFLAGS   =  -DWITHGPU -march=native -m64 -mssse3 -Wno-write-strings -g -I. -I$(CUDA)/include $(PYBIND_FLAGS)
else
CXXFLAGS   =  -DWITHGPU -march=native -m64 -mssse3 -Wno-write-strings -O3 -I. -I$(CUDA)/include $(PYBIND_FLAGS)
endif
LFLAGS     =  -lgmp -lpthread -L$(CUDA)/lib64 -lcudart
else
ifdef debug
CXXFLAGS   =  -march=native -m64 -mssse3 -Wno-write-strings -g -I. -I$(CUDA)/include $(PYBIND_FLAGS)
else
CXXFLAGS   =  -march=native -m64 -mssse3 -Wno-write-strings -O3 -I. -I$(CUDA)/include $(PYBIND_FLAGS)
endif
LFLAGS     =  -lgmp -lpthread
endif

#--------------------------------------------------------------------

ifdef gpu
ifdef debug
$(OBJDIR)/GPU/GPUEngine.o: GPU/GPUEngine.cu
	$(NVCC) -allow-unsupported-compiler -G -maxrregcount=0 --ptxas-options=-v --compile --compiler-options -fPIC -ccbin $(CXXCUDA) -m64 -g -I$(CUDA)/include -gencode=arch=compute_$(ccap),code=sm_$(ccap) -o $(OBJDIR)/GPU/GPUEngine.o -c GPU/GPUEngine.cu
else
$(OBJDIR)/GPU/GPUEngine.o: GPU/GPUEngine.cu
	$(NVCC) -allow-unsupported-compiler -maxrregcount=0 --ptxas-options=-v --compile --compiler-options -fPIC -ccbin $(CXXCUDA) -m64 -O2 -I$(CUDA)/include -gencode=arch=compute_$(ccap),code=sm_$(ccap) -o $(OBJDIR)/GPU/GPUEngine.o -c GPU/GPUEngine.cu
endif
endif

$(OBJDIR)/%.o : %.cpp
	$(CXX) $(CXXFLAGS) -o $@ -c $<

all: Rotor

Rotor: $(OBJET)
	@echo Making Rotor...
	$(CXX) $(OBJET) $(LFLAGS) $(PYBIND_FLAGS) -o Rotor

rotor.so: $(OBJET) $(OBJDIR)/PythonEndings.o
	$(CXX) $(OBJET) $(OBJDIR)/PythonEndings.o $(LFLAGS) $(PYBIND_FLAGS) -o rotor$(shell python3-config --extension-suffix)

$(OBJET): | $(OBJDIR) $(OBJDIR)/GPU $(OBJDIR)/hash

$(OBJDIR):
	mkdir -p $(OBJDIR)

$(OBJDIR)/GPU: $(OBJDIR)
	cd $(OBJDIR) &&	mkdir -p GPU

$(OBJDIR)/hash: $(OBJDIR)
	cd $(OBJDIR) &&	mkdir -p hash

clean:
	@echo Cleaning...
	@rm -f obj/*.o
	@rm -f obj/GPU/*.o
	@rm -f obj/hash/*.o
	@rm -rf rotor.so