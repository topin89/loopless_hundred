# Loopless Hundred
That's how I can  print 1 to 100 in C++ without a loop, goto or recursion

A long time ago, I saw a [question on Quora](https://www.quora.com/How-can-I-print-1-to-100-in-C++-without-a-loop-goto-or-recursion), "How can I print 1 to 100 in C++ without a loop, goto or recursion?". I saw a lot of good answers, but the unspoken truth was "you don't".

Even simplest answers didn't account loops in `printf` or `std::cout`, not to mention all the underlying kernel code. And even without that code, there is always c++ runtime for static constructor and destructor. C runtime for _data_ and _bss_ sections initialization.

The only way I saw to actually do that was to take a microcontroller and load some C++ code stripped of all the runtime. Time passed, I was bored on a New Year Holidays, and this was born. I used [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html) 1.1.0 on Windows, but I'm sure you can run this code on any supported OS (That is, Mac and Linux). The board I used is STM32F4DISCOVERY, and MCU is STM32F407VGT6. Those 1MB of internal flash sure come in handy.

This code has no recursions, loops or gotos, and also it's assembly listing has only forward unconditional branches. About as loopless as it can be.

To do that, I disabled all runtime functions in the linker, and also I edited startup assembly code, replaced first function in the vector table from `Reset_handler` to `main`. Stack register setups by MCU anyway, so it works not by some miracle. `Default_Handler` and `Reset_Hanler` were replaced with single `nop`. They won't be called anyway. In linker script entry point was also replaced with `main`.

And to make sure no branching backward at all, every function was marked as `always_inline`. Not needed to answer the question, but I want it anyway.

To "print" numbers, tm1637 display was used. All wait cycles like `Hal_Delay` was replaced with `TIM2` Update events enabled, but not NVIC interrupt, and `wfe` asm opcode, with _SEVONPEND_ bit set. That means no actual interrupts was used, so no random branching. Just because I can.

Resulting binary is about 500KB, but it's indeed have no backward braches in direct or indirect way.
