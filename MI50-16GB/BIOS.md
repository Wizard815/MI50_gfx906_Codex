# MI50 16GB VBIOS: images, flashing, risks, recovery

[Back to README](README.md). Source numbers like [S2] refer to [SOURCES.md](SOURCES.md).

For the 32 GB card (V420, 274474, 275395, Apple Pro VII 32GB) use the
[evilJazz gist](https://gist.github.com/evilJazz/14a4c82a67f2c52a6bb5f9cea02f5e13) and
the [main README](../README.md). This page covers **16 GB-specific** images only.

> **Never flash a 32 GB ROM onto a 16 GB card, or a 16 GB ROM onto a 32 GB card.** The
> memory configuration is part of the image. The ROMs are signed (RSA-checked at
> boot), so you cannot edit one to fit [S2][S26].

## Known 16 GB images

Only versions and part numbers seen in a source are listed. TechPowerUp's VBIOS pages
were behind a captcha during this research, so the TechPowerUp details below come from
search-result snippets and the gist comment, not from opening the pages directly.

| Image | P/N | Version | MCLK | PCIe | Power cap | UEFI / ReBAR | Display | Status |
|---|---|---|---|---|---|---|---|---|
| **MI50 16GB stock (server)** | `113-D1631400-X11` | `016.004.000.056.013521` | 600 MHz per comment; TPU lists 930/1000 MHz core/mem | Gen4 | ~225 W? (unconfirmed) | unknown | no | Version confirmed by 2 sources (TPU entry compiled 2021-05-12 [S1], gist comment [S2]). Everything else is one report |
| Radeon Pro VII 16GB (AMD workstation) | `113-D1640600-104` | `016.004.000.064.015319` | 1000 MHz | Gen3 max | 198 W | UEFI yes | not stated | one report [S2]; config string `AMD_VEGA20_D16406_GLXE_A1_HBM2_16GB_WS`; MD5 `64d1c521a9fd0ae594e4ca9d9e14f8c7` |
| Radeon Pro VII 16GB (Lenovo OEM) | `113-D1640700-100` | `016.004.000.064.015933` | 1000 MHz | Gen3 max | 350 W HW max, 198 W driver default | UEFI yes, ReBAR yes, FP64 1/2 | not stated | one report [S2], TPU entry #277846; MD5 `bd58215956ab3f4e069476078c6f1e70`; the commenter's top pick for inference; overdrive off by default |
| Radeon VII (consumer, "MOONSHOT" final) | `113-D3600200-106` | `016.004.000.038.011717` | 1000 MHz | Gen3 | n/a | no UEFI on early reference BIOS [S27] | **conflicting reports** | see below |
| Radeon VII (Dell Alienware final) | `113-D3600400-106` | `016.004.000.040.011848` | | | | | conflicting | one report [S2] |
| Radeon VII pre-release builds | `113-D3600110-026`, `113-D3600200-105`, `113-D3600400-105` | `016.004.000.026.011383`, `.030.011639`, `.030.011641` | | | | | | listed for research only; don't flash [S2] |

The commenter tested on an original AMD PCB under CachyOS, ROCm 7.2.4 and Mesa 26.1.2.
They report FP64 fully active on the Pro VII BIOS and about 62–65 tok/s with Ollama
Vulkan (model not stated) [S2].

### Which BIOS gives display output?

- **Stock MI50 16GB:** no display output (legacy-only video at most) [S2][S25].
- **Radeon VII BIOS:** the reports conflict.
  - **No display:** the gist comment says the consumer Radeon VII BIOS gives no working display on the MI50 16GB because of a connector-table mismatch ("research pending") [S2]. The same comment says the Radeon VII BIOS disables FP64 [S2].
  - **Display works:** Chinese buyers and sellers say the main benefit of the Radeon VII flash is working video output, with standard Adrenalin or PRO drivers [S21].
  - A possible explanation is that Chinese-market boards are wired differently from original AMD boards. That is **unverified**. Another TechPowerUp poster says only "Chinese MI50/Radeon VII" boards accept Radeon VII / Pro VII BIOSes (one report) [S3].
- **Radeon Pro VII BIOS:** one article says this cross-flash "reportedly enables display output" [S25]. The gist comment lists no display result for the 16 GB Pro VII images. Unverified.
- For an LLM server, display output doesn't matter. The ML350 has its own iLO and onboard VGA.

### What each cross-flash changes (16 GB)

| Target BIOS | Gains | Losses / risks |
|---|---|---|
| Pro VII (Lenovo/WS) | 1000 MHz MCLK if the stock BIOS really runs 600 MHz; UEFI; ReBAR (Lenovo); 1/2-rate FP64 kept | PCIe Gen3 max (one report); 198 W default cap; one user bricked a 16 GB card with a Pro VII image [S3] |
| Radeon VII | consumer driver support on Windows; display on some boards; this is the BIOS Chinese sellers pre-flash | FP64 cut to 1/4; PCIe Gen3; fan curve meant for a 3-fan card (the passive card has no fan header in use); may report `sramecc-` and break stock rocBLAS (inference, see [UPGRADES-AND-TUNING.md](UPGRADES-AND-TUNING.md#common-problems)) |
| Stock `113-D1631400-X11` | PCIe Gen4; matches AMD support | no display; MCLK question unresolved |

**Fan control:** the MI50 is passive, so no BIOS gives you fan control of a fan that isn't
there. Fans you add have to be driven by the motherboard or a script (see
[UPGRADES-AND-TUNING.md](UPGRADES-AND-TUNING.md)). On the MI25 a WX9100 BIOS enables the
card's own fan header [S28], but nobody has reported the same on the MI50.

## Flashing with amdvbflash (Linux)

amdvbflash is the former `atiflash`. A Linux build is mirrored at stylesuxx/amdvbflash,
after AMD had the original repository removed [S26]. Versions matter:

- amdvbflash **3.31** detected an MI50 16GB but failed to flash it. Version **5.0**
  flashed it, after which the card reported as "Radeon VII Pro/MI50 32GB" and 3.31 could no
  longer see it (one report) [S3].
- `amdvbflash_linux_4.71` allowed overrides where 4.104E did not, on an MI25 (one report) [S29].

Commands from the gist [S2] (same for 16 and 32 GB; adapter `0` is an example, so check `-i` first):

```bash
sudo ./amdvbflash -i                     # list adapters, note index, SSID, P/N, version
sudo ./amdvbflash -ai                    # more detail
sudo ./amdvbflash -s 0 backup_16gb.rom   # BACK UP FIRST ("0x100000 bytes saved")
md5sum backup_16gb.rom                   # store it off the server too
sudo ./amdvbflash -f -fp -fa -fv -p 0 target.rom
# success ends with "Restart System To Complete VBIOS Update." -> full power cycle
```

| Flag | Meaning [S2] |
|---|---|
| `-p <n> <file>` | program adapter n with file |
| `-s <n> <file>` | save adapter n ROM to file |
| `-f` | force; skip SSID / AsicID / boot-card checks. "Not supported in external versions" per the mirror README [S26] |
| `-fp` | bypass the BIOS P/N check |
| `-fa` | bypass the "already programmed" check |
| `-fv` | bypass the newer-version check |

Checklist:

1. Check `md5sum` of the target ROM against a published hash (MD5s above, or the gist's 32 GB table).
2. Flash **one card at a time**, with the other GPUs either the same model or pulled, so `-p 0` can't hit the wrong card.
3. If you get "Adapter not found", reports say disabling Secure Boot fixed it, and so did running with CSM off, ReBAR on and Above 4G on [S2].
4. `sysfs` ROM reads return only about 104 KB, so always dump with `amdvbflash -s` [S2].
5. Use the same BIOS on every card in a ROCm box. The gist author saw ROCm problems with mixed VBIOS versions (one report, 32 GB) [S2].

## Risks

- **Signature or ID lockout.** After a Pro VII flash, one user found amdvbflash refused to write other images ("hardware ID and signature mismatch") and could not restore the original [S3].
- **Wrong capacity reported.** The same 16 GB card reported as 32 GB after the flash [S3]. Don't run an image whose memory configuration doesn't match the card.
- **No POST.** A Dell Precision 7820 would not boot or show video with an MI50 installed (stock BIOS, display on another card) [S18]. An HP Z840 needed Above 4G Decoding to POST [S30].
- **Thermals after a flash.** Radeon VII / Pro VII images expect a fan or blower cooler. A passive MI50 relies entirely on chassis airflow, and the junction throttles hard above ~95 °C (1700 → ~860 MHz, one report) [S2].
- **Driver support.** One Chinese doc notes the Radeon Pro VII Windows driver stopped receiving updates around Aug 2023 (one report) [S21]. This is irrelevant on Linux.

## Recovery

Neither the gist nor the threads give a tested MI50 recovery procedure. Here is what the sources suggest:

1. **Software first.** If amdvbflash still lists the card, flash your backup with `-f -fp -fa -fv -p`. Try another amdvbflash version (3.x / 4.71 / 5.0), since they behave differently [S3][S29].
2. **Boot from another GPU.** Boot with the ML350's onboard VGA so the bricked card isn't needed for POST, then flash it as a secondary adapter.
3. **External SPI programmer.** On TechPowerUp the advice for a card amdvbflash can't reverse was "reflash the VBIOS with a hardware programmer" [S3]. On the MI25, people use a SOIC-8 clip with a Raspberry Pi or CH341A on the BIOS chip [S28]. Nobody has documented the MI50's chip location or whether it is write-protected (unverified). Read and save the chip's current contents before writing.
4. A dual-BIOS switch on the MI50 has **not** been reported in any source. Don't count on having one.
