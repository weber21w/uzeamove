###############################################################################
# Makefile for the project SpriteDemo
###############################################################################

## General Flags
PROJECT = Uze-A-Move
GAME= Uze-A-Move
MCU = atmega644
TARGET = $(GAME).elf
CC = avr-gcc
INFO=../gameinfo.properties
UZEBIN_DIR = ../../../bin

## Kernel settings
KERNEL_DIR = ../../../kernel
KERNEL_OPTIONS  = -DVIDEO_MODE=3 -DINTRO_LOGO=0
KERNEL_OPTIONS += -DTILE_WIDTH=8 -DTILE_HEIGHT=12
KERNEL_OPTIONS += -DVRAM_TILES_H=30 -DVRAM_TILES_V=18 #was 30x18
KERNEL_OPTIONS += -DSCREEN_TILES_V=17 #was 18
KERNEL_OPTIONS += -DSCROLLING=0 -DSOUND_MIXER=1 #-DRESOLUTION_EXT=1
KERNEL_OPTIONS += -DMAX_SPRITES=14 -DRAM_TILES_COUNT=24

## Options common to compile, link and assembly rules
COMMON = -mmcu=$(MCU)

## Compile options common for all C compilation units.
CFLAGS = $(COMMON)
CFLAGS += -Wall -gdwarf-2 -std=gnu99 -DF_CPU=28636360UL -Os -fsigned-char -ffunction-sections 
CFLAGS += -MD -MP -MT $(*F).o -MF dep/$(@F).d 
CFLAGS += $(KERNEL_OPTIONS)


## Assembly specific flags
ASMFLAGS = $(COMMON)
ASMFLAGS += $(CFLAGS)
ASMFLAGS += -x assembler-with-cpp -Wa,-gdwarf2

## Linker flags
LDFLAGS = $(COMMON)
LDFLAGS += -Wl,-Map=$(GAME).map 
LDFLAGS += -Wl,-gc-sections 


## Intel Hex file production flags
HEX_FLASH_FLAGS = -R .eeprom

HEX_EEPROM_FLAGS = -j .eeprom
HEX_EEPROM_FLAGS += --set-section-flags=.eeprom="alloc,load"
HEX_EEPROM_FLAGS += --change-section-lma .eeprom=0 --no-change-warnings


## Objects that must be built in order to link
OBJECTS = uzeboxVideoEngineCore.o uzeboxCore.o uzeboxSoundEngine.o uzeboxSoundEngineCore.o uzeboxVideoEngine.o $(GAME).o 

## Objects explicitly added by the user
LINKONLYOBJECTS = 

## Include Directories
INCLUDES = -I"$(KERNEL_DIR)" 

## Included data files
DATA_FILES = ../data/bg.inc ../data/sprites.inc ../data/title.inc

## Build
all: $(TARGET) $(GAME).hex $(GAME).eep $(GAME).lss $(GAME).uze

../data/bg.inc: ../data/bg.png ../data/bg.gconvert.xml
	$(UZEBIN_DIR)/gconvert ../data/bg.gconvert.xml

../data/sprites.inc: ../data/sprites.png ../data/sprites.gconvert.xml
	$(UZEBIN_DIR)/gconvert ../data/sprites.gconvert.xml

../data/title.inc: ../data/title.png ../data/title.gconvert.xml
	$(UZEBIN_DIR)/gconvert ../data/title.gconvert.xml

## Compile Kernel files
uzeboxVideoEngineCore.o: $(KERNEL_DIR)/uzeboxVideoEngineCore.s
	$(CC) $(INCLUDES) $(ASMFLAGS) -c  $<

uzeboxSoundEngineCore.o: $(KERNEL_DIR)/uzeboxSoundEngineCore.s
	$(CC) $(INCLUDES) $(ASMFLAGS) -c  $<

uzeboxCore.o: $(KERNEL_DIR)/uzeboxCore.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

uzeboxSoundEngine.o: $(KERNEL_DIR)/uzeboxSoundEngine.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

uzeboxVideoEngine.o: $(KERNEL_DIR)/uzeboxVideoEngine.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

## Compile game sources
$(GAME).o: ../$(GAME).c $(DATA_FILES)
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

##Link
$(TARGET): $(OBJECTS)
	 $(CC) $(LDFLAGS) $(OBJECTS) $(LINKONLYOBJECTS) $(LIBDIRS) $(LIBS) -o $(TARGET)

%.hex: $(TARGET)
	avr-objcopy -O ihex $(HEX_FLASH_FLAGS)  $< $@

%.eep: $(TARGET)
	-avr-objcopy $(HEX_EEPROM_FLAGS) -O ihex $< $@ || exit 0

%.lss: $(TARGET)
	avr-objdump -h -S $< > $@

%.uze: $(TARGET)
	$(UZEBIN_DIR)/packrom $(GAME).hex $@ $(INFO)

UNAME := $(shell sh -c 'uname -s 2>/dev/null || echo not')
AVRSIZEFLAGS := -A ${TARGET}
ifneq (,$(findstring MINGW,$(UNAME)))
AVRSIZEFLAGS := -C --mcu=${MCU} ${TARGET}
endif

size: ${TARGET}
	@echo
	@avr-size -C --mcu=${MCU} ${TARGET}

## Clean target
.PHONY: clean
clean:
	-rm -rf $(OBJECTS) $(GAME).* dep/* *.uze $(DATA_FILES)


## Other dependencies
-include $(shell mkdir dep 2>/dev/null) $(wildcard dep/*)

