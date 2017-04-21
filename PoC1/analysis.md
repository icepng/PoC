PoDoFo 0.9.5 function TextExtractor::ExtractText in TextExtractor.cpp:77 cause a NULL pointer dereference

## Analyzer
code from https://sourceforge.net/p/podofo/code/HEAD/tree/podofo/trunk/  (2017-04-09)

compile: 
``` bash
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=/home/icepng/aaaa/ -DCMAKE_BUILD_TYPE=DEBUG -DCMAKE_EXE_LINKER_FLAGS="-fsanitize=address -fno-omit-frame-pointer" ../
make -j
```
and run it:
``` bash
./podofo-code/podofo/trunk/build/tools/podofotxtextract/podofotxtextract ./PoC
```

## Crash Info
``` bash
=================================================================
==12966==ERROR: AddressSanitizer: SEGV on unknown address 0xbebec0bd (pc 0x080f4c21 bp 0xbf8fcbc8 sp 0xbf8fcbc0 T0)
    #0 0x80f4c20 in PoDoFo::PdfVariant::DelayedLoad() const /home/icepng/podofo-code/podofo/trunk/podofo/base/../../src/base/PdfVariant.h:545
    #1 0x80f4c8f in PoDoFo::PdfVariant::GetReal() const /home/icepng/podofo-code/podofo/trunk/src/base/PdfVariant.h:675
    #2 0x80f38af in TextExtractor::ExtractText(PoDoFo::PdfMemDocument*, PoDoFo::PdfPage*) /home/icepng/podofo-code/podofo/trunk/tools/podofotxtextract/TextExtractor.cpp:77
    #3 0x80f36a5 in TextExtractor::Init(char const*) /home/icepng/podofo-code/podofo/trunk/tools/podofotxtextract/TextExtractor.cpp:48
    #4 0x80f6ceb in main /home/icepng/podofo-code/podofo/trunk/tools/podofotxtextract/podofotxtextract.cpp:52
    #5 0xb6a9b636 in __libc_start_main (/lib/i386-linux-gnu/libc.so.6+0x18636)
    #6 0x80f3470  (/home/icepng/podofo-code/podofo/trunk/build/tools/podofotxtextract/podofotxtextract+0x80f3470)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /home/icepng/podofo-code/podofo/trunk/podofo/base/../../src/base/PdfVariant.h:545 PoDoFo::PdfVariant::DelayedLoad() const
==12966==ABORTING
```

## analysis
``` c++
if( eType == ePdfContentsType_Keyword )
        {
            // support 'l' and 'm' tokens
            if( strcmp( pszToken, "l" ) == 0 ||  // l: line to 
                strcmp( pszToken, "m" ) == 0 )  // m: move to 
            {
                dCurPosX = stack.top().GetReal();
                stack.pop();
                dCurPosY = stack.top().GetReal();
                stack.pop();
            }
......
else if ( eType == ePdfContentsType_Variant )
        {
            stack.push( var );
        }
```
I set the breakpoints and I found there had 45 ePdfContentsType_Variant but 23 MoveTo Tokens.
so stack.top() will be NULL in "dCurPosY = stack.top().GetReal();", then cause a NULL point dereference.
So the upstream don't give strict check about Path Construction Operators "MoveTo".