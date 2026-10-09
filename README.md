# voidbr-limine-config

Configuração do bootloader [Limine](https://github.com/limine-bootloader/limine) para o VoidBR Linux.

O pacote gera o `limine.conf` automaticamente, copia kernel e initramfs para a ESP e aplica o tema VoidBR (Tokyo Night). Também regenera a configuração sempre que um kernel é instalado, atualizado ou removido.

## O que o pacote instala

| Arquivo | Função |
|---|---|
| `/usr/bin/update-limine` | Gera o `limine.conf` e copia kernel/initramfs para a ESP |
| `/usr/local/bin/update-limine` | Link para `/usr/bin/update-limine` |
| `/etc/default/limine` | Configurações do usuário (timeout, cmdline, tema) |
| `/etc/kernel.d/post-install/90-limine` | Hook de kernel do Void: roda o `update-limine` ao instalar, atualizar ou reconfigurar kernel (`xbps-reconfigure -f linuxX.Y`), depois do dracut |
| `/etc/kernel.d/post-remove/90-limine` | Hook de kernel do Void: roda o `update-limine` ao remover kernel |
| `/etc/xbps.d/hooks.d/95-limine-update.hook` | Hook do xbps: roda o `update-limine` quando um pacote instala ou atualiza um `/boot/vmlinuz-*` (só kernel) |
| `/boot/efi/limine/voidbr-tokyonight.png` | Wallpaper do menu |

Os binários do Limine não vêm no pacote: na instalação/upgrade eles são copiados do pacote `limine` (`/usr/share/limine/`) para a ESP, sempre na versão instalada:

| Origem | Destino |
|---|---|
| `/usr/share/limine/BOOTX64.EFI` | `/boot/efi/EFI/limine/BOOTX64.EFI` |
| `/usr/share/limine/BOOTX64.EFI` | `/boot/efi/EFI/BOOT/BOOTX64.EFI` (só se não existir ou já for do Limine) |
| `/usr/share/limine/limine-bios.sys` | `/boot/efi/limine/limine-bios.sys` |

## Uso

```sh
sudo update-limine
```

O script precisa rodar como root. Ele:

1. lê `/etc/default/limine` (se existir) por cima dos valores padrão;
2. aborta se `/boot/efi` estiver no `/etc/fstab` mas não estiver montada;
3. escolhe os kernels de `/boot` do mais novo ao mais antigo, enquanto couberem na ESP;
4. copia para `/boot/efi/limine/` só o que mudou e remove os kernels que saíram;
5. grava `/boot/efi/limine/limine.conf` com uma entrada normal e uma de recuperação (`single`, sem `quiet`/`splash`/plymouth) por kernel, além de Memtest86+ (se houver `/boot/memtest.bin`), Configurações de Firmware UEFI e Windows Boot Manager (só em UEFI; o Windows só aparece se o `efibootmgr` o listar).

Em btrfs com subvolumes, o `rootflags=subvol=...` é adicionado sozinho. Se o `voidbr-snapper-manager` estiver instalado, entra também o submenu de snapshots.

O `limine.conf` é gerado: não edite à mão, mude o `/etc/default/limine` e rode `update-limine` de novo.

## Configuração (`/etc/default/limine`)

| Variável | Padrão | Descrição |
|---|---|---|
| `LIMINE_TIMEOUT` | `5` | Segundos até bootar a entrada padrão |
| `LIMINE_VERBOSE` | `no` | Saída detalhada do Limine (`yes`/`no`) |
| `LIMINE_DISTRO_NAME` | `VoidBR` | Nome usado no título das entradas |
| `LIMINE_CMDLINE_LINUX_DEFAULT` | `rw loglevel=4` | Parâmetros do kernel (o `root=UUID=` é adicionado sozinho) |
| `LIMINE_TERM_PALETTE` | Tokyo Night | 8 cores separadas por `;` |
| `LIMINE_TERM_PALETTE_BRIGHT` | Tokyo Night | 8 cores "bright" separadas por `;` |
| `LIMINE_TERM_BACKGROUND` | `ffffffff` | Fundo do terminal (`TTRRGGBB`) |
| `LIMINE_TERM_FOREGROUND` | `c0caf5` | Cor do texto |
| `LIMINE_TERM_BACKGROUND_BRIGHT` | `ffffffff` | Fundo "bright" |
| `LIMINE_TERM_FOREGROUND_BRIGHT` | `c0caf5` | Texto "bright" |
| `LIMINE_WALLPAPER` | `voidbr-tokyonight.png` | Arquivo dentro de `/boot/efi/limine/` |
| `LIMINE_WALLPAPER_STYLE` | `stretched` | `stretched`, `centered` ou `tiled` |
| `LIMINE_BRANDING` | `VoidBR Linux` | Texto no topo do menu (vazio = não mostra) |
| `LIMINE_BRANDING_COLOR` | `7aa2f7` | Cor do texto do topo (`RRGGBB`) |

O arquivo está em `backup=()`: numa atualização, o xbps mantém a versão editada e grava a nova ao lado como `.new-<versão>`.

## Empacotamento

O pacote é gerado pelo [voidbr-pkgmake](https://github.com/voidlinuxbr/voidbr-pkgmake) a partir de `pkgfile/PKGFILE`.

Na instalação e no upgrade, o `post_install` copia os binários do Limine para a ESP e roda o `update-limine`. Durante o build da ISO (`MKISO_BUILD`) ele não faz nada; dentro do instalador (`VOIDBR_INSTALLER`) copia os binários, mas deixa o `update-limine` para o instalador.

## Licença

MIT — veja [LICENSE](LICENSE).
