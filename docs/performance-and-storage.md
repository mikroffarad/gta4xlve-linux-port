# Перенесення Mobian на UFS і WineASIO

## UFS — виконано

Користувач підтвердив, що Android і його дані не потрібні. Відформатовано лише userdata /dev/sda31 (49 GiB) в ext4; таблицю розділів і решту firmware-розділів не змінено. SD /dev/mmcblk2p2 збережено.

- Новий PARTUUID: 1b81e7e6-f50d-419b-a739-2aeef8da3335.
- Новий UUID ext4: faa6abb5-636f-4fb5-b6bc-e1f5167674f5.
- rsync -aHAXx --numeric-ids переніс систему зі збереженням власників, ACL, xattrs і hardlinks. Фінальна синхронізація виконана після зупинки GDM/Wine.
- fstab оновлено лише в новій системі; попередня версія — /etc/fstab.sd-backup.
- Створено каталоги /dev, /proc, /sys, /run, /tmp; для /tmp встановлено 1777.
- e2fsck -f -n пройшов усі п’ять стадій без помилок.
- Поточний boot прочитано повністю (96 MiB) і збережено як boot-original.img. У boot-ufs.img змінено лише PARTUUID у cmdline (30 байтів відрізняються). Kernel і DTB залишені без змін.
- Запис нового boot перевірено побайтово. Після перезавантаження findmnt / підтвердив /dev/sda31; приблизно 18 GiB зайнято, 30–31 GiB доступно.
- Wi-Fi/USB-мережа, GDM, PipeWire, WirePlumber активні.
- Wine dosdevices/e:: оновлено з /dev/mmcblk2p2 на /dev/disk/by-partlabel/userdata; попереднє посилання збережене в ~/ufs-migration/wine-e-device.sd-backup.

Безпечний тест послідовного читання до форматування: dd if=DEVICE of=/dev/null bs=4M count=64 iflag=direct. SD 85.3 MB/s, UFS 394 MB/s (4.62x). Це один 256 MiB зразок; він не вимірює випадковий I/O, запис, швидкість запуску FL чи DSP.

Копії boot також є на планшеті в /home/mobian/ufs-migration. SHA256:

- original: 6aca50ce14f8e738a4a9379d1f22572917e82bb861bd204f0303a900f232cbff
- UFS: ad8329b73d9a038866620e5caa8973a0c2972100e2a9cbb8506b654d37bcd6ca

Для повернення завантаження із SD, за наявності SD та доступу до працюючого Linux на планшеті: перевірити відповідність boot alias /dev/sda19, потім записати boot-original.img у /dev/disk/by-partlabel/boot і перевірити cmp перед перезавантаженням. Якщо Linux не завантажується, потрібне відновлення boot через Samsung Download Mode. SD сама по собі не перемикає bootloader назад.

## WineASIO — журнал підготовки

Upstream wineasio commit b5e668103ad13e6f51f4118ed7090592213e5ca2. Зібрано на x86_64 ПК з headers/import libraries/tools установленої на планшеті Wine 11.17. Виявлено дві проблеми збірки:

1. winegcc цього SDK за замовчуванням використовує clang/lld. Розташування .init після .text спричиняло переповнення unsigned RVA і виклик DllMain за адресою +4 GiB. Перекомпонування з -fuse-ld=bfd усунуло цю помилку.
2. Для fake PE модуля потрібно явно передати winebuild -F wineasio64.dll; інакше stub просить wineasio.dll замість wineasio64.dll.

Після виправлень regsvr32 успішний і тест CoCreateInstance=0. Backup початкових system.reg/user.reg: ~/wineasio-stage/backup (це копія файлів активного Wine, а не гарантовано узгоджений знімок всього префікса).

Встановлено pipewire-jack:arm64 та pipewire-jack:amd64 (залежності останнього ~25 MB). Для Box64 0.3.4 потрібен саме x86_64 libjack PipeWire через BOX64_LD_LIBRARY_PATH; native pw-jack сам по собі не замінює emulated JACK.

Подальший тест виявив відсутні у Box64 0.3.4 libc wrappers pidfd_open, pidfd_send_signal, mallinfo2. Підготовлена окрема upstream-збірка Box64 commit 5524b6203372600f30cb2ab71f84b1e5c553fb1f, ARM_DYNAREC=ON; системний пакет /usr/bin/box64 не замінюється.

Джерела: https://github.com/wineasio/wineasio ; https://docs.pipewire.org/page_man_pw-jack_1.html ; https://docs.pipewire.org/page_man_pipewire_1.html

## Завантаження: прибрано зайве очікування мережі

Після переходу на UFS systemd-analyze показав 126.922 s до завершення старту, з них 120.216 s — systemd-networkd-wait-online. networkctl показав усі інтерфейси unmanaged, а мережа фактично під керуванням NetworkManager. Вимкнено лише systemd-networkd-wait-online.service; NetworkManager-wait-online залишено увімкненим. Після наступного перезавантаження 2026-09-11: 1.758 s kernel + 13.442 s userspace = 15.201 s, graphical.target 13.439 s. Це окреме виправлення очікування, не доказ, що сам UFS прискорив boot у 8 разів. Відкат: sudo systemctl enable systemd-networkd-wait-online.service.

Окремий Box64 встановлено в /opt/box64-fl/bin/box64, версія 0.4.5, commit 5524b6203372600f30cb2ab71f84b1e5c553fb1f. Після збірки перепов’язано з libm Debian trixie 2.41: host cross-toolchain інакше вимагав GLIBC_2.43/2.44, яких немає на планшеті. У фінальному бінарнику таких version requirements немає; --version запускається на планшеті. /usr/bin/box64 залишено пакетним 0.3.4.

## WineASIO: функціональний тест 2026-09-11

Виявлено ще одну причину падіння: Init() WineASIO викликає mlockall(MCL_FUTURE), а невеликий memlock limit призводив до `failed to map segment from shared object` при завантаженні наступних бібліотек. Для процесів спеціального запуску встановлено `ulimit -l 0`, щоб цей необов’язковий виклик не блокував усі подальші mappings. Це не глобальне налаштування і не заборона свопу всій системі; ціною є відсутність блокування аудіопам’яті в цьому процесі.

Повний беззвучний тест власним ASIO host через Box64 0.4.5/Wine 11.17:

```
CoCreateInstance=0
inputs=0 outputs=2 buffer=512 sample_rate=48000
CreateBuffers=0
Start=0
callbacks_in_10s=934
Stopped
Disposed
exit=0
```

Очікувана частота callbacks — 48000/512=93.75 Hz. Тест підтверджує створення COM-об’єкта, запуск callback-потоку, зупинку, звільнення буферів і завершення процесу. Він не доводить нуль underruns або низьку фізичну затримку: у короткому pw-top після тестів sink мав ERR=3; окремого приросту на стабільному проєкті ще не виміряно.

`/usr/local/bin/flstudio` тепер використовує перевірені параметри:

- PATH=/opt/box64-fl/bin:$PATH (системний Box64 не замінено).
- BOX64_LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu/pipewire-0.3/jack.
- WINEDLLPATH=/opt/wine-11.17-amd64-wow64/lib/wine/x86_64-unix.
- ulimit -l 0.
- WINEASIO_NUMBER_INPUTS=0, WINEASIO_NUMBER_OUTPUTS=2, WINEASIO_AUTOSTART_SERVER=off.
- WINEASIO_PREFERRED_BUFFERSIZE=512, PIPEWIRE_LATENCY=512/48000 (можна перевизначати для тесту).
- PIPEWIRE_LOG_SYSTEMD=false, DISABLE_RTKIT=1 — лише в клієнті FL, не в системному PipeWire. RTKit у цьому запуску не підвищує пріоритет; це робоча конфігурація для перевірки сумісності, не остаточне налаштування мінімальної затримки.

Оригінальний launcher збережено як /usr/local/bin/flstudio.pre-wineasio. Той самий новий запуск також доступний як /usr/local/bin/flstudio-wineasio. Файл flstudio-wineasio поруч із цим звітом — точна встановлена версія.
