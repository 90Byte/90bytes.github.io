---
title: "Post: Cosmopolitan libc"
last_modified_at: 2021-06-16T12:58:39+0900
categories:
  - Blog
featuredPost: true
tags:
  - Multiplatform
---

I found an interesting project on GitHub and want to share it.

Around the time you learn the basics of computer science, you understand that the execution process of a program
involves the following: a program stored in secondary storage is divided into TEXT, DATA, STACK, and HEAP segments to
become a process that's loaded into main memory. The process starts at an entry point, executes machine code
instructions, and returns to the OS upon completion. But given that Linux and Windows, or architectures like AMD64 and
i386, use the same CPU, their machine codes are the same. So why do operating systems require separate programs?

1. The structure of the programs differs. Windows uses the Portable Executable (PE) format, while Linux uses the
   Executable and Linkable Format (ELF).
2. The operation of loaders and linkers, which load the program and dynamically link it, differs.
3. The implementation of libc, the standard C library, is different.

For points 1 and 2, someone has [neatly summarized the information](http://tmmse.xyz/2020/04/22/linuxandwindows/).

The Cosmopolitan project originated from the realization that the PE header used in Windows programs can be encoded into
a Unix 6th edition shell script. Including the 5 files provided by this project during the build process of a C program
ensures the executable runs on Linux, Mac, or Windows, and even on bare metal. The Windows PE header starts with 'MZ',
which is executed by being recognized as a Unix shell script, and directly mapped into memory through a separate link
script.

The Hello World example provided on Cosmopolitan's [GitHub](https://github.com/jart/cosmopolitan) is as follows:

```shell
wget https://justine.lol/cosmopolitan/cosmopolitan-amalgamation-1.0.zip unzip cosmopolitan-amalgamation-1.0.zip printf 'main() { printf("hello world\\n"); }\n' >hello.c gcc -g -Os -static -nostdlib -nostdinc -fno-pie -no-pie -mno-red-zone \ 
     -fno-omit-frame-pointer -pg -mnop-mcount \ 
     -o hello.com.dbg hello.c -fuse-ld=bfd -Wl,-T,ape.lds \ 
     -include cosmopolitan.h crt.o ape.o cosmopolitan.a objcopy -S -O binary hello.com.dbg hello.com
```

Lines 1 and 2 download and unzip the zip file containing the amalgamated[^1] cosmopolitan libc and object files. Line 3
simply places a basic C Hello World example into hello.c. Notice the lack of includes mentioned later. The reason it's
not int main() is due to C89, where not defining a return type defaults to int. This syntax was removed in C99.

Then, it's built using gcc. The -nostdlib and -nostdinc flags mean that standard libraries and headers from the system
directories are not used during the link and preprocess steps, respectively. This is because the cosmopolitan libc
replaces the existing libc. -fuse-ld is used when linking with a linker different from the default. -T is a flag used
for linking with a separate script, part of which looks like the code snippet shown.

```ld
ENTRY(_start)
PHDRS {
 Head PT_LOAD FLAGS(5);
 Rom PT_LOAD FLAGS(5);
 Ram PT_LOAD FLAGS(6);
 stack PT_GNU_STACK FLAGS(6);
}
SECTIONS {
 .head SEGMENT_START("text-segment", 0x400000) : AT(0x2000) {
 HIDDEN(_base = .);
 KEEP(*(.head))
 KEEP(*(.text.head))
 . = ALIGN(8);
 HIDDEN(ape_phdrs = .);
 KEEP(*(.elf.phdrs))
 HIDDEN(ape_phdrs_end = .);
 . = ALIGN(8);
 HIDDEN(ape_note = .);
 KEEP(*(.note.openbsd.ident))
 KEEP(*(.note.netbsd.ident))
 HIDDEN(ape_note_end = .);
 KEEP(*(.pe.header))
 HIDDEN(ape_pe_sections = .);
 KEEP(*(.pe.sections))
 HIDDEN(ape_pe_sections_end = .);
 KEEP(*(.macho))
 . = ALIGN(8);
 HIDDEN(ape_macho_end = .);
 KEEP(*(.ape.pad.head))
 . = ALIGN(((255 & 4) == 4) || ((255 & 2) == 2) ? 0x1000 : 16);
 HIDDEN(_ehead = .);
 } :Head
 .text . : {
 *(.text.real)
 KEEP(*(SORT_BY_NAME(.sort.text.real.*)))
 HIDDEN(_ereal = .);
 . += 1;
 *(.start)
 KEEP(*(.initprologue))
 KEEP(*(SORT_BY_NAME(.init.*)))
 KEEP(*(.init))
 KEEP(*(.initepilogue))
 KEEP(*(.pltprologue))
 *(.plt)
 KEEP(*(.pltepilogue))
 KEEP(*(.pltgotprologue))
 *(.plt.got)
 KEEP(*(.pltgotepilogue))
 *(.text.startup .text.startup.*)
 *(.text.exit .text.exit.*)
 *(.text.unlikely .text.*_unlikely .text.unlikely.*)
 *(SORT_BY_ALIGNMENT(.text.antiquity))
 *(SORT_BY_ALIGNMENT(.text.antiquity.*))
 KEEP(*(.textwindowsprologue))
 *(.text.windows)
 KEEP(*(.textwindowsepilogue))
 *(SORT_BY_ALIGNMENT(.text.modernity))
 *(SORT_BY_ALIGNMENT(.text.modernity.*))
 *(SORT_BY_ALIGNMENT(.text.hot))
 *(SORT_BY_ALIGNMENT(.text.hot.*))
 KEEP(*(.keep.text))
 *(.text .stub .text.*)
 KEEP(*(SORT_BY_NAME(.sort.text.*)))
 KEEP(*(.ape.pad.test));
 *(.test.unlikely)
 *(.test .test.*)
 KEEP(*(.ape.pad.privileged));
 . += . > 0 ? 1 : 0;
 HIDDEN(__privileged_start = .);
 . += . > 0 ? 1 : 0;
 *(.privileged)
 HIDDEN(__privileged_end = .);
 . += . > 0 ? 1 : 0;
 KEEP(*(.ape.pad.rodata));
 *(.rodata .rodata.*)
 *(.ubsan.types)
 *(.ubsan.data)
 KEEP(*(.commentprologue))
 KEEP(*(.comment))
 KEEP(*(.commentepilogue))
 KEEP(*(.idata.ro));
 KEEP(*(SORT_BY_NAME(.idata.ro.*)))
 . = ALIGN(8);
 PROVIDE_HIDDEN(__init_array_start = .);
 KEEP(*(SORT_BY_INIT_PRIORITY(.init_array.*)
 SORT_BY_INIT_PRIORITY(.ctors.*)))
 KEEP(*(.ctors))
 KEEP(*(.init_array))
 KEEP(*(.preinit_array))
 PROVIDE_HIDDEN(__init_array_end = .);
 . = ALIGN(8);
 PROVIDE_HIDDEN(__fini_array_start = .);
 KEEP(*(SORT_BY_INIT_PRIORITY(.fini_array.*)
 SORT_BY_INIT_PRIORITY(.dtors.*)))
 KEEP(*(.dtors))
 PROVIDE_HIDDEN(__fini_array_end = .);
 KEEP(*(.initroprologue))
 KEEP(*(SORT_BY_NAME(.initro.*)))
 KEEP(*(.initroepilogue))
 KEEP(*(SORT_BY_NAME(.sort.rodata.*)))
 KEEP(*(.ape.pad.text))
 . = ALIGN(0x1000);
 HIDDEN(_etext = .);
 PROVIDE_HIDDEN(etext = .);
 } :Rom
 .data . : {
 KEEP(*(.dataprologue))
 *(.data .data.*)
 KEEP(*(SORT_BY_NAME(.sort.data.*)))
 . += . > 0 ? 1 : 0;
 KEEP(*(.gotprologue))
 *(.got)
 KEEP(*(.gotepilogue))
 KEEP(*(.gotpltprologue))
 *(.got.plt)
 KEEP(*(.gotpltepilogue))
 . = ALIGN(8);
 KEEP(*(SORT_BY_NAME(.piro.relo.sort.*)))
 PROVIDE_HIDDEN(__relo_end = .);
 . = ALIGN(8);
 KEEP(*(SORT_BY_NAME(.piro.data.sort.*)))
 KEEP(*(.piro.pad.data))
 . = ALIGN(0x1000);
 HIDDEN(_edata = .);
 PROVIDE_HIDDEN(edata = .);
 } :Ram
 .zip . : {
 KEEP(*(SORT_BY_NAME(.zip.*)))
 HIDDEN(_ezip = .);
 }
 .bss ALIGN(8) : {
 KEEP(*(SORT_BY_NAME(.piro.bss.init.*)))
 *(.piro.bss)
 KEEP(*(SORT_BY_NAME(.piro.bss.sort.*)))
 HIDDEN(__piro_end = .);
 . += . > 0 ? 1 : 0;
 *(SORT_BY_ALIGNMENT(.bss))
 *(SORT_BY_ALIGNMENT(.bss.*))
 *(COMMON)
 KEEP(*(SORT_BY_NAME(.sort.bss.*)))
 . = ALIGN(0x10000);
 HIDDEN(_end = .);
 PROVIDE_HIDDEN(end = .);
 }
 .shstrtab : { *(.shstrtab) }
 .strtab : { *(.strtab) }
 .symtab : { *(.symtab) }
 .stab 0 : { *(.stab) }
 .stabstr 0 : { *(.stabstr) }
 .stab.excl 0 : { *(.stab.excl) }
 .stab.exclstr 0 : { *(.stab.exclstr) }
 .stab.index 0 : { *(.stab.index) }
 .stab.indexstr 0 : { *(.stab.indexstr) }
 .comment 0 : { *(.comment) }
 .debug 0 : { *(.debug) }
 .line 0 : { *(.line) }
 .debug_srcinfo 0 : { *(.debug_srcinfo) }
 .debug_sfnames 0 : { *(.debug_sfnames) }
 .debug_aranges 0 : { *(.debug_aranges) }
 .debug_pubnames 0 : { *(.debug_pubnames) }
 .debug_info 0 : { *(.debug_info .gnu.linkonce.wi.*) }
 .debug_abbrev 0 : { *(.debug_abbrev) }
 .debug_line 0 : { *(.debug_line .debug_line.* .debug_line_end ) }
 .debug_frame 0 : { *(.debug_frame) }
 .debug_str 0 : { *(.debug_str) }
 .debug_loc 0 : { *(.debug_loc) }
 .debug_macinfo 0 : { *(.debug_macinfo) }
 .debug_weaknames 0 : { *(.debug_weaknames) }
 .debug_funcnames 0 : { *(.debug_funcnames) }
 .debug_typenames 0 : { *(.debug_typenames) }
 .debug_varnames 0 : { *(.debug_varnames) }
 .debug_pubtypes 0 : { *(.debug_pubtypes) }
 .debug_ranges 0 : { *(.debug_ranges) }
 .debug_macro 0 : { *(.debug_macro) }
 .debug_addr 0 : { *(.debug_addr) }
 .gnu.attributes 0 : { KEEP(*(.gnu.attributes)) }
 .GCC.command.line 0 : { *(.GCC.command.line) }
 /DISCARD/ : {
 *(__mcount_loc)
 *(.discard)
 *(.yoink)
 *(.*)
 }
}
```

From what I understood, it seems to play a role in mapping due to different memory maps across OSes... Though it seems
crucial, I lack the expertise to fully grasp it and will need to review it carefully later.

Running a program built on Linux shows that it doesn't immediately run on my Mac with the default zsh shell or my Linux
that's been switched to zsh. It runs normally with bash or sh.

Moreover, a program once run seems to integrate itself as a regular program of that platform, no longer running on other
platforms but executing faster and more efficiently from the next run onward.

Wondering what can be accomplished using cosmopolitan, I tried building existing C programs with it. Thinking of
CPython, I ran configure and blindly added options to the Makefile before running make...

The first error I encountered, unsurprisingly, involved system headers searched by #include<>. After scripting to
comment out all #include<> lines and executing, I ran into an error about not finding the pthread_key_t type.

[cosmopolitan libc](https://github.com/jart/cosmopolitan/blob/master/libc/isystem/pthread.h) does contain types like
pthread_once_t included in sys/types.h, but why other pthread-related types are missing remains unclear...

The values for pthread appear to vary, with some being uint and others int. Most seem to correspond with pthread_t, so I
typedef'd it as int, as defined by cosmopolitan.

After that, for some reason, the PyAPI_FUNC macro doesn't work... It seems related to the link options, but as I've yet
to fully understand those gcc options, I'll need to revisit them.

Although my lack of related knowledge might have hindered a proper explanation, I introduced it as an interesting
project. I think it'd be a good study to slowly delve into low-level concepts like memory maps, as my understanding
remains textbook-level.

[^1]: amalgamation means bundling libraries into one header for easier use.