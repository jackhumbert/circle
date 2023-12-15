cmake_minimum_required(VERSION 3.16)

set(CMAKE_OSX_SYSROOT "")

project(circle C CXX ASM)

set(RASPPI 3 CACHE STRING "Raspberry Pi Version")
set(AARCH 64 CACHE STRING "Raspberry Pi Bits")

set(CMAKE_SYSTEM_NAME Generic)
SET(CMAKE_SYSTEM_VERSION 1)
set(CMAKE_SYSTEM_PROCESSOR aarch64)
set(CMAKE_CROSSCOMPILING TRUE)

unset(_CMAKE_APPLE_ARCHS_DEFAULT)

set(CMAKE_C_COMPILER "/opt/homebrew/bin/aarch64-none-elf-gcc")
set(CMAKE_CXX_COMPILER "/opt/homebrew/bin/aarch64-none-elf-g++")
set(CMAKE_AR "/opt/homebrew/bin/aarch64-none-elf-ar")
set(CMAKE_LINKER "/opt/homebrew/bin/aarch64-none-elf-ld")
set(CMAKE_OBJCOPY "/opt/homebrew/bin/aarch64-none-elf-objcopy")

set(CMAKE_C_FLAGS "")
set(CMAKE_CXX_FLAGS "")
set(CMAKE_CXX_LINK_FLAGS "")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -T ${CMAKE_CURRENT_SOURCE_DIR}/circle.ld")

if(RASPPI EQUAL 1)
    set(MCPU arm1176jzf-s)
    set(TARGET kernel)
endif()
if(RASPPI EQUAL 2)
    set(MCPU cortex-a7)
    set(TARGET kernel7)
endif()
if(RASPPI EQUAL 3)
    set(MCPU cortex-a53)
    if(AARCH EQUAL 64)
        set(TARGET kernel8)
    else()
        set(TARGET kernel8-32)
    endif()
endif()
if(RASPPI EQUAL 4)
    set(MCPU cortex-a72)
    if(AARCH EQUAL 64)
        set(TARGET kernel8-rpi4)
    else()
        set(TARGET kernel7l)
    endif()
endif()
set(TARGET ${TARGET} PARENT_SCOPE)

file(GLOB_RECURSE SOURCES "lib/*.cpp")
file(GLOB_RECURSE S_SOURCES "lib/*.S")

if(NOT RASPPI EQUAL 4)
    list(REMOVE_ITEM SOURCES 
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/bcm54213.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/bcmpciehostbridge.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/bcmrandom200.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/interruptgic.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/dma4channel.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/devicetreeblob.cpp

        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhcicommandmanager.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhcidevice.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhciendpoint.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhcieventmanager.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhcimmiospace.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhciring.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhciroothub.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhcirootport.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhcisharedmemallocator.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhcislotmanager.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/xhciusbdevice.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/usbaudiocontrol.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/usbaudiostreaming.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/usbaudiofunctopology.cpp

        ${CMAKE_CURRENT_SOURCE_DIR}/lib/sound/usbsoundbasedevice.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/sound/usbsoundcontroller.cpp
    )
endif()

if(RASPPI EQUAL 4)
    list(REMOVE_ITEM SOURCES 
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/bcmrandom.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/interrupt.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/mphi.cpp

        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/dwhcidevice.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/dwhciframeschednper.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/dwhciframeschednsplit.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/dwhciframeschedper.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/dwhcirootport.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/dwhcixactqueue.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/dwhcicompletionqueue.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/dwhcixferstagedata.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/usb/dwhciframeschediso.cpp
    )
endif()

if(AARCH EQUAL 64)
    list(REMOVE_ITEM SOURCES
    # 32
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/exceptionhandler.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/memory.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/pagetable.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/startup.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/synchronize.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/libhelper.cpp
    )
    list(REMOVE_ITEM S_SOURCES 
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/exceptionstub.S
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/cache-v7.S
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/startup.S
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/libstub.S
    )
endif()

list(APPEND SOURCES
${CMAKE_CURRENT_SOURCE_DIR}/addon/linux/sprintf.cpp
${CMAKE_CURRENT_SOURCE_DIR}/addon/linux/pthread.cpp
${CMAKE_CURRENT_SOURCE_DIR}/addon/linux/kthread.cpp
${CMAKE_CURRENT_SOURCE_DIR}/addon/linux/mutex.cpp

${CMAKE_CURRENT_SOURCE_DIR}/addon/fatfs/diskio.cpp
${CMAKE_CURRENT_SOURCE_DIR}/addon/fatfs/ff.c
${CMAKE_CURRENT_SOURCE_DIR}/addon/fatfs/ffsystem.cpp
${CMAKE_CURRENT_SOURCE_DIR}/addon/fatfs/ffunicode.c

${CMAKE_CURRENT_SOURCE_DIR}/addon/SDCard/emmc.cpp
${CMAKE_CURRENT_SOURCE_DIR}/addon/SDCard/mmchost.cpp
${CMAKE_CURRENT_SOURCE_DIR}/addon/SDCard/sdhost.cpp
)

add_library(circle STATIC ${SOURCES} ${S_SOURCES} ${CMAKE_CURRENT_SOURCE_DIR}/addon/lvgl/lvgl.cpp)

set_target_properties(circle PROPERTIES LINK_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/circle.ld)

target_include_directories(circle PUBLIC 
    ${CMAKE_CURRENT_SOURCE_DIR}/include 
    ${CMAKE_CURRENT_SOURCE_DIR}/addon 
    ${CMAKE_CURRENT_SOURCE_DIR}/app/lib 
    ${CMAKE_CURRENT_SOURCE_DIR}/addon/vc4 
    ${CMAKE_CURRENT_SOURCE_DIR}/addon/vc4/interface/khronos/include
)

target_compile_options(circle PUBLIC
    -fno-exceptions 
    $<$<COMPILE_LANGUAGE:CXX>:-fno-rtti>
    $<$<COMPILE_LANGUAGE:CXX>:-nostdinc++>
    -mcpu=${MCPU}
    -mlittle-endian
    -Wall
    -fsigned-char
    -ffreestanding
    -g
    $<IF:$<STREQUAL:${AARCH},64>,-mstrict-align,-mno-unaligned-access>
    -U__unix__
    -U__linux__
    -O2
    $<$<COMPILE_LANGUAGE:CXX>:-std=c++14>
    $<$<COMPILE_LANGUAGE:CXX>:-Wno-aligned-new>
    -Wno-parentheses
    # --specs=nosys.specs
    # -ffreestanding 
    # -flto
    # -fno-use-cxa-atexit
)

target_compile_definitions(circle PUBLIC
    __circle__=450301
    RASPPI=${RASPPI}
    AARCH=${AARCH}
    STDLIB_SUPPORT=1
    __VCC0REVER__=0x04000000
    DEFAULT_KEYMAP="US"
    USB_GADGET_VENDOR_ID=0xFEED
    KERNEL_MAX_SIZE=0x8000000
    ARM_ALLOW_MULTI_CORE
    # LEAVE_QEMU_ON_HALT
    # KERNEL_STACK_SIZE=0x800000
    # NDEBUG
    # _GLIBCXX_INCLUDE_NEXT_C_HEADERS
)

target_include_directories(circle PUBLIC
    /Users/jackhumbert/Github/circle-stdlib/install/aarch64-none-circle/include
)

target_link_libraries(circle PUBLIC
    gcc
    # m
    # c
    # /Users/jackhumbert/Github/circle-stdlib/install/aarch64-none-circle/lib/libc.a
    # /Users/jackhumbert/Github/circle-stdlib/install/aarch64-none-circle/lib/libg.a
    # /Users/jackhumbert/Github/circle-stdlib/install/aarch64-none-circle/lib/libm.a
)

target_link_options(circle PUBLIC
    -Wl,--section-start=.init=$<IF:$<STREQUAL:${CMAKE_SYSTEM_PROCESSOR},aarch64>,0x80000,0x8000>
    # -Wl,-flto 
    # -ffreestanding
    -nostdlib
    # -specs=nano.specs
    # -specs=nosys.specs
    # -Wl,/Applications/ArmGNUToolchain/13.2.Rel1/aarch64-none-elf/bin/../lib/gcc/aarch64-none-elf/13.2.1/../../../../aarch64-none-elf/lib/libc.a
    # -Wl,/Applications/ArmGNUToolchain/13.2.Rel1/aarch64-none-elf/bin/../lib/gcc/aarch64-none-elf/13.2.1/libgcc.a
    # /Applications/ArmGNUToolchain/13.2.Rel1/aarch64-none-elf/bin/../lib/gcc/aarch64-none-elf/13.2.1/../../../../aarch64-none-elf/lib/libstdc++.a
    # /Applications/ArmGNUToolchain/13.2.Rel1/aarch64-none-elf/bin/../lib/gcc/aarch64-none-elf/13.2.1/../../../../aarch64-none-elf/lib/libm.a
    -T ${CMAKE_CURRENT_SOURCE_DIR}/circle.ld
)