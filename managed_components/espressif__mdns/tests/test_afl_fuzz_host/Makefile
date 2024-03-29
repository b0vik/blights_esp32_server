TEST_NAME=test
FUZZ=afl-fuzz
COMPONENTS_DIR=$(IDF_PATH)/components
COMPILER_ICLUDE_DIR=$(shell echo `which xtensa-esp32-elf-gcc | xargs dirname | xargs dirname`/xtensa-esp32-elf)

CFLAGS=-g -Wno-unused-value -Wno-missing-declarations -Wno-pointer-bool-conversion -Wno-macro-redefined -Wno-int-to-void-pointer-cast -DHOOK_MALLOC_FAILED -DESP_EVENT_H_ -D__ESP_LOG_H__ \
                 -I. -I../.. -I../../include -I../../private_include -I ./build/config  \
                 -I$(COMPONENTS_DIR) \
                 -I$(COMPONENTS_DIR)/driver/include \
                 -I$(COMPONENTS_DIR)/esp_common/include \
                 -I$(COMPONENTS_DIR)/esp_event/include \
                 -I$(COMPONENTS_DIR)/esp_eth/include \
                 -I$(COMPONENTS_DIR)/esp_hw_support/include \
                 -I$(COMPONENTS_DIR)/esp_netif/include \
                 -I$(COMPONENTS_DIR)/esp_netif/private_include \
                 -I$(COMPONENTS_DIR)/esp_netif/lwip \
                 -I$(COMPONENTS_DIR)/esp_rom/include \
                 -I$(COMPONENTS_DIR)/esp_system/include \
                 -I$(COMPONENTS_DIR)/esp_timer/include \
                 -I$(COMPONENTS_DIR)/esp_wifi/include \
                 -I$(COMPONENTS_DIR)/freertos/FreeRTOS-Kernel \
                 -I$(COMPONENTS_DIR)/freertos/FreeRTOS-Kernel/include \
                 -I$(COMPONENTS_DIR)/freertos/esp_additions/include/freertos \
                 -I$(COMPONENTS_DIR)/hal/include \
                 -I$(COMPONENTS_DIR)/hal/esp32/include \
                 -I$(COMPONENTS_DIR)/heap/include \
                 -I$(COMPONENTS_DIR)/log/include \
                 -I$(COMPONENTS_DIR)/lwip/lwip/src/include \
                 -I$(COMPONENTS_DIR)/linux/include \
                 -I$(COMPONENTS_DIR)/lwip/port/esp32/include \
                 -I$(COMPONENTS_DIR)/lwip/lwip/src/include/lwip/apps \
                 -I$(COMPONENTS_DIR)/soc/include \
                 -I$(COMPONENTS_DIR)/soc/esp32/include \
                 -I$(COMPONENTS_DIR)/soc/src/esp32/include \
                 -I$(COMPONENTS_DIR)/xtensa/include \
                 -I$(COMPONENTS_DIR)/xtensa/esp32/include \
                 -I$(COMPILER_ICLUDE_DIR)/include


MDNS_C_DEPENDENCY_INJECTION=-include mdns_di.h
ifeq ($(MDNS_NO_SERVICES),on)
    CFLAGS+=-DMDNS_NO_SERVICES
endif

ifeq ($(INSTR),off)
    CC=gcc
    CFLAGS+=-DINSTR_IS_OFF
    TEST_NAME=test_sim
else
    CC=afl-clang-fast
endif
CPP=$(CC)
LD=$(CC)
OBJECTS=esp32_mock.o mdns.o test.o esp_netif_mock.o

OS := $(shell uname)
ifeq ($(OS),Darwin)
  LDLIBS=
else
   LDLIBS=-lbsd
   CFLAGS+=-DUSE_BSD_STRING
endif

all: $(TEST_NAME)

%.o: %.c
	@echo "[CC] $<"
	@$(CC) $(CFLAGS) -c $< -o $@

mdns.o: ../../mdns.c
	@echo "[CC] $<"
	@$(CC) $(CFLAGS) -include mdns_mock.h $(MDNS_C_DEPENDENCY_INJECTION) -c $< -o $@

$(TEST_NAME): $(OBJECTS)
	@echo "[LD] $@"
	@$(LD)  $(OBJECTS) -o $@ $(LDLIBS)

fuzz: $(TEST_NAME)
	@$(FUZZ) -i "in" -o "out" -- ./$(TEST_NAME)

clean:
	@rm -rf *.o *.SYM $(TEST_NAME) out
