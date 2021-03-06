# ------------------------------------------------
# Generic Makefile for Synopsys Designware ARC processor
# ------------------------------------------------

######################################
# target
######################################
TARGET = TencentOS_tiny

APPLICATION = hello_world

######################################
# building variables
######################################
# debug build?
DEBUG = 1
# optimization
OPT = -O0

# the root of TencentOS Tiny
TOP_DIR = ../../..

#######################################
# paths
#######################################
# Build path
BUILD_DIR = build

#######################################
# embarc_bsp relative conifg
#######################################

BOARD ?= nsim
BD_VER ?= 10
CUR_CORE ?= arcem
TOOLCHAIN ?= mw
OLEVEL ?= O0
OUT_DIR_ROOT ?= $(BUILD_DIR)

export BOARD
export BD_VER
export CUR_CORE
export TOOLCHAIN
export OUT_DIR_ROOT
export OLEVEL

EMBARC_BSP_ROOT = $(TOP_DIR)/platform/vendor_bsp/Synopsys/ARC/embarc_bsp

EMBARC_BSP_TEST_FILE = $(EMBARC_BSP_ROOT)/arc/startup/arc_startup.s

ifeq ($(wildcard $(EMBARC_BSP_TEST_FILE)),)
$(info )
$(info )
$(info )
$(info ######################################################################################################)
$(info #                                                                                                    #)
$(info #    embarc_bsp not exist!!!                                                                         #)
$(info #    please git embarc_bsp from github                                                               #)
$(info #    run the following cmd in folder <TOP_DIR>/platform/vendor_bsp/Synopsys/ARC>:                             #)
$(info #    git clone https://github.com/foss-for-synopsys-dwc-arc-processors/embarc_bsp.git -b upstream    #)
$(info #                                                                                                    #)
$(info ######################################################################################################)
$(info )
$(info )
$(info )
$(error [Makefile.error]: embarc_bsp not exist!!!)
endif

EMBARC_ROOT = $(EMBARC_BSP_ROOT)
export EMBARC_ROOT

EMBARC_OUT_DIR = $(OUT_DIR_ROOT)/obj_$(BOARD)_$(BD_VER)/$(TOOLCHAIN)_$(CUR_CORE)

ifeq ($(TOOLCHAIN),mw)
COMPILER_ARG_FILE := $(EMBARC_OUT_DIR)/embARC_generated/ccac.arg
LDF_FILE := $(EMBARC_OUT_DIR)/linker_mw.ldf
MDB_ARG_FILE := $(EMBARC_OUT_DIR)/embARC_generated/mdb.arg
else
COMPILER_ARG_FILE := $(EMBARC_OUT_DIR)/embARC_generated/gcc.arg
LDF_FILE := $(EMBARC_OUT_DIR)/linker_gnu.ldf
NSIM_PROPS := $(EMBARC_OUT_DIR)/embARC_generated/nsim.props
endif

######################################
# source
######################################
# C sources
KERNEL_SRC =  \
		${wildcard $(TOP_DIR)/kernel/core/*.c}
C_SOURCES += $(KERNEL_SRC)

ARCH_SRC = \
		${wildcard $(TOP_DIR)/arch/arc/arcem/*.c} \
		${wildcard $(TOP_DIR)/arch/arc/common/*.c}
C_SOURCES += $(ARCH_SRC)

CMSIS_SRC =  \
		${wildcard $(TOP_DIR)/osal/cmsis_os/cmsis_os.c}
C_SOURCES += $(CMSIS_SRC)

APPLICATION_SRC =  \
		${wildcard $(TOP_DIR)/board/ARC_NSIM_EM/BSP/Src/*.c} \
		${wildcard $(TOP_DIR)/examples/$(APPLICATION)/*.c}

C_SOURCES += $(APPLICATION_SRC)

# ASM sources
ASM_SOURCES =  \
		$(TOP_DIR)/arch/arc/arcem/port_s.s

ifeq ($(TOOLCHAIN),mw)
PREFIX =
CC = $(PREFIX)ccac
AS = $(PREFIX)ccac

SZ = $(PREFIX)sizeac

OBJCOPY	= $(PREFIX)elf2bin
ELF2HEX	= $(PREFIX)elf2hex

ELF2HEX_INOPT = -Q -I
ELF2HEX_OUTOPT = -o
ELF2BIN_OPT =
else
PREFIX = arc-elf32-
CC = $(PREFIX)gcc
AS = $(PREFIX)gcc -x assembler-with-cpp
SZ = $(PREFIX)size
OBJCOPY = $(PREFIX)objcopy
ELF2HEX	= $(OBJCOPY)

ELF2HEX_INOPT = -O ihex
ELF2HEX_OUTOPT =
ELF2BIN_OPT = -O binary -S
endif

#######################################
# CFLAGS
#######################################

# C defines

NSIM_STACK_CHECK := -DARC_FEATURE_STACK_CHECK=0
# 1: enable hardware stack check, 0 disable hardware stack check

ifeq ($(TOOLCHAIN),mw)
TOOLCHAIN_DEF := -D__MW__
else
TOOLCHAIN_DEF := -D__GNU__ -D_HAVE_LIBGLOSS_
endif

C_DEFS =  \
	$(TOOLCHAIN_DEF) \
	$(NSIM_STACK_CHECK) \
	-DBOARD_NSIM \
	-DCURRENT_CORE=$(CUR_CORE) \
	-DEMBARC_TCF_GENERATED \
	-DHW_VERSION=$(BD_VER) \
	-DLIB_CONSOLE \
	-DLIB_CLIB \
	-D_HEAPSIZE=8192 \
	-D_HOSTLINK_ \
	-D_NSIM_ \
	-D_STACKSIZE=2048 \
	-DARC_FEATURE_STACK_CHECK=0

# include dirs
ARCH_INC = \
		-I $(TOP_DIR)/arch/arc/arcem \
		-I $(TOP_DIR)/arch/arc/common/include
C_INCLUDES += $(ARCH_INC)

BOARD_INC = \
		-I $(TOP_DIR)/board/ARC_NSIM_EM/BSP/Inc
C_INCLUDES += $(BOARD_INC)

KERNEL_INC = \
		-I $(TOP_DIR)/kernel/core/include \
		-I $(TOP_DIR)/kernel/pm/include \
		-I $(TOP_DIR)/kernel/hal/include
C_INCLUDES += $(KERNEL_INC)

CMSIS_INC = \
		-I $(TOP_DIR)/osal/cmsis_os
C_INCLUDES += $(CMSIS_INC)

EMBARC_BSP_INC =  \
		-I $(EMBARC_BSP_ROOT)/board \
		-I $(EMBARC_BSP_ROOT)/board/nsim/configs/10 \
		-I $(EMBARC_BSP_ROOT)/include \
		-I $(EMBARC_BSP_ROOT)/include/arc \
		-I $(EMBARC_BSP_ROOT)/include/device/designware \
		-I $(EMBARC_BSP_ROOT)/include/device/subsystem \
		-I $(EMBARC_BSP_ROOT)/library \
		-I $(EMBARC_OUT_DIR)/embARC_generated
C_INCLUDES += $(EMBARC_BSP_INC)

ifeq ($(TOOLCHAIN),mw)
ASFLAGS = @$(COMPILER_ARG_FILE) -Hnoccm -Hnosdata -Wincompatible-pointer-types -Hnocopyr -Hasmcpp -$(OLEVEL) -g $(C_DEFS) $(C_INCLUDES) -MMD -MT $@ -MF $@.d

CFLAGS =  @$(COMPILER_ARG_FILE) -Hnoccm -Hnosdata -Wincompatible-pointer-types -Hnocopyr -Hnocplus -$(OLEVEL) -g $(C_DEFS) $(C_INCLUDES) -MMD -MT $@ -MF $@.d

LDFLAGS = -Hhostlink @$(COMPILER_ARG_FILE) -Hnocopyr -Hnosdata -Hnocrt -Hldopt=-Coutput=$(BUILD_DIR)/mw_arcem.map -Hldopt=-Csections -Hldopt=-Ccrossfunc -Hldopt=-Csize -zstdout $(LDF_FILE)
else

ASFLAGS = @$(COMPILER_ARG_FILE) -fdata-sections -ffunction-sections -mno-sdata -$(OLEVEL) -g $(C_DEFS) $(C_INCLUDES) -x assembler-with-cpp

CFLAGS =  @$(COMPILER_ARG_FILE) -fdata-sections -ffunction-sections -mno-sdata -$(OLEVEL) -g $(C_DEFS) $(C_INCLUDES) -std=gnu99

LDFLAGS = --specs=nsim.specs @$(COMPILER_ARG_FILE) -mno-sdata -nostartfiles -Wl,-M,-Map=$(BUILD_DIR)/gnu_arcem.map -lm -Wl,--script=$(LDF_FILE)
endif
# default action: build all
all: $(BUILD_DIR)/$(TARGET).elf $(BUILD_DIR)/$(TARGET).hex $(BUILD_DIR)/$(TARGET).bin


#######################################
# build the application
#######################################
# list of objects
OBJECTS = $(addprefix $(BUILD_DIR)/,$(notdir $(C_SOURCES:.c=.o)))
vpath %.c $(sort $(dir $(C_SOURCES)))
# list of ASM program objects
OBJECTS += $(addprefix $(BUILD_DIR)/,$(notdir $(ASM_SOURCES:.s=.o)))
vpath %.s $(sort $(dir $(ASM_SOURCES)))
OBJECTS += $(addprefix $(BUILD_DIR)/,$(notdir $(ASM_SOURCES_S:.S=.o)))
vpath %.S $(sort $(dir $(ASM_SOURCES_S)))

# EMBARC_PREBUILT_LIBRARY = -Hldopt=-Bgrouplib
EMBARC_PREBUILT_LIBRARY = $(EMBARC_OUT_DIR)/libembarc.a


build_embarc_lib: | $(BUILD_DIR)
	make -f $(EMBARC_BSP_ROOT)/options/options.mk V=1 embarc_lib

$(BUILD_DIR)/%.o: %.c Makefile | $(BUILD_DIR)
	$(CC) -c $(CFLAGS) $< -o $@

$(BUILD_DIR)/%.o: %.s Makefile | $(BUILD_DIR)
	$(AS) -c $(ASFLAGS) $< -o $@

$(BUILD_DIR)/%.o: %.S Makefile | $(BUILD_DIR)
	$(AS) -c $(ASFLAGS) $< -o $@

$(BUILD_DIR)/$(TARGET).elf: build_embarc_lib $(OBJECTS) Makefile
	$(CC) $(LDFLAGS) $(OBJECTS) $(EMBARC_PREBUILT_LIBRARY) -o $@
	$(SZ) $@

$(BUILD_DIR)/%.hex: $(BUILD_DIR)/%.elf | $(BUILD_DIR)
	$(ELF2HEX) $(ELF2HEX_INOPT) $< $(ELF2HEX_OUTOPT) $@

$(BUILD_DIR)/%.bin: $(BUILD_DIR)/%.elf | $(BUILD_DIR)
	$(OBJCOPY) $(ELF2BIN_OPT) $< $@

$(BUILD_DIR):
	mkdir $@

#######################################
# clean up
#######################################
clean:
	-rm -fR $(BUILD_DIR)

nsim_debug : $(BUILD_DIR)/$(TARGET).elf
	mdb -nooptions -nogoifmain -toggle=include_local_symbols=1 -nsim -off=binary_stdin -off=binary_stdout  -on=load_at_paddr -on=reset_upon_restart -off=flush_pipe -off=cr_for_more -OKN @$(MDB_ARG_FILE) $<
#######################################
# dependencies
#######################################
run : $(BUILD_DIR)/$(TARGET).elf
ifeq ($(TOOLCHAIN),mw)
	mdb -nooptions -nogoifmain -toggle=include_local_symbols=1 -nsim -off=binary_stdin -off=binary_stdout  -on=load_at_paddr -on=reset_upon_restart -off=flush_pipe -off=cr_for_more -OKN @$(MDB_ARG_FILE) -run $<
else
	nsimdrv -p nsim_emt=1 -propsfile $(NSIM_PROPS) $<
endif

-include $(wildcard $(BUILD_DIR)/*.d)

# *** EOF ***