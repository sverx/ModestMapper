# the ModestMapper

the ModestMapper is an open _low-cost_ **rewritable** cartridge for the SEGA Master System.

It's based on [doragasu's FrugalMapper cartridge](https://gitlab.com/doragasu/sms-sl2map) and shares many of its features:

- uses a Flash memory chip instead of ROM chips, thus being rewritable
- uses only discrete logic chips, as few of them as possible
- can be programmed and reprogrammed after having being assembled
- supports up to 4 Mbit (512 KiB) but can use 2 Mbit (256 KiB) and 1 Mbit (128 KiB) Flash chips too (currently _untested_ but the design allows for that)

While doragasu's FrugalMapper supports slot-2 SEGA mapper only, the ModestMapper supports both slot-1 **and** slot-2 SEGA mapper, making it perfect for [devkitSMS](https://github.com/sverx/devkitSMS) games that use [transparent code bankswitching](https://github.com/sverx/devkitSMS#advanced-how-to-use-more-than-32-kib-of-code-in-your-smsggsgsc-rom-banked-code) for example, but can be used for any game requiring slot-1 and slot-2 paging (please read the note on the _Words of caution_ section below).

# How it works

the ModestMapper implements only a subset of the complete [SEGA mapper](https://www.smspower.org/Development/Mappers#TheSegaMapper) specifications so:

- slot-0 paging isn't supported, thus the first 16 KiB of the data in the ROM will always be accessible at address $0000-$3FFF
- slot-1 and slot-2 paging **are** supported, thus any of the 16 KiB blocks of the ROM can be accessible at addresses $4000-$7FFF (slot-1) and $8000-$BFFF (slot-2)
- up to 32 × 16-KiB ROM pages are supported
- advanced SEGA mapper features such as bankshifting or write protection aren't supported

# Words of caution

Since address decoding is partial, any writes to addresses $E000-$FFFF (RAM _mirror_ addresses) will work as writes to the SEGA mapper ports $FFFC-$FFFF - see the following table:

       writes to                              |     effect
    ------------------------------------------+----------------------
    %111x xxxx xxxx xx00 ($FFFC and mirrors)  |   _none_ (ignored)
    %111x xxxx xxxx xx01 ($FFFD and mirrors)  |   _none_ (ignored)
    %111x xxxx xxxx xx10 ($FFFE and mirrors)  |   **set slot-1 page**
    %111x xxxx xxxx xx11 ($FFFF and mirrors)  |   **set slot-2 page**

This shouldn't cause any issues provided that the program **never** writes to RAM _mirror_ addresses ([Emulicious](https://emulicious.net)' debugger has a very useful feature to detect if this happens).

Also of note is that the slot-1 and slot-2 pages are **NOT** initialized by the hardware so the [SEGA signature](https://www.smspower.org/Development/ROMHeader) that's **required** by the Master System BIOS to be in the ROM at a certain locations **can't** be located at $7FF0 but **has** to be located at $3FF0 instead (devkitSMS has a macro for that: *SMS_EMBED_SEGA_ROM_HEADER_16KB*) - or at $1FF0.

# How to write a ROM onto the cartridge

You can use either [Raphnet's SMS cartridge reader/programmer](https://www.raphnet-tech.com/products/sms_cartridge_reader_programmer/index.php) with an [updated firmware](https://github.com/sverx/smscprogr) (tested personally) or you can use [doragasu's MegaWiFi programmer](https://gitlab.com/doragasu/mw-prog) (not tested yet but it should work as the ModestMapper is compatible with the FrugalMapper by design).

# License

the ModestMapper cartridge design is Open Hardware, licensed under the CERN-OHL-S license, as it's a  derivate work of [doragasu's FrugalMapper cartridge](https://gitlab.com/doragasu/sms-sl2map)

# Acknowledgements

This project wouldn't have ever been possible without doragasu's FrugalMapper and doragasu's invaluable help while designing this. Also a big THANK YOU to the [SMS Power!](https://www.smspower.org) community and in particolar to @undeveloper ❤️

