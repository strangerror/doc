# Извлечение проприетарных файлов из прошивки

По сути, это вольный перевод статьи из [wiki LineageOS](https://wiki.lineageos.org/extracting_blobs_from_zips).


Сначала нужно скачать файл полной прошивки для установки через recovery. Важно - полной прошивки, НЕ ОТА!!!<br>
Для 8T: https://bigota.d.miui.com/V12.5.6.0.RCXMIXM/miui_WILLOWGlobal_V12.5.6.0.RCXMIXM_b9c4b22e3f_11.0.zip<br>
Для 8: https://bigota.d.miui.com/V12.5.12.0.RCOEUXM/miui_GINKGOEEAGlobal_V12.5.12.0.RCOEUXM_f9fdc6c783_11.0.zip<br>
Обновления для устройств представляют собой `split block-based OTA`.

Для извлечения нужны скрипты, входящие в состав среды сборки прошивки. В данном примере рассмотрены LineageOS и crDroid.<br>
Создаём каталог, в котором будем выполнять все операции и переходим в него. Дальнейшие действия выполняются в созданном каталоге.:
```sh
mkdir ~/AndroidBuild/ginkgo_dump && cd $_
```
Скачиваем всё нужное для получения дампа.<br>
Сами скрипты для извлечения:
```sh
git clone https://github.com/LineageOS/android_tools_extract-utils tools/extract-utils
```
Исходники для нашего устройства. Т.к. для ginkgo/willow последняя версия дерева устройства "lineage-19.1", то и качаем той же версии: 
```sh
git clone -b lineage-19.1 https://github.com/LineageOS/android_device_xiaomi_ginkgo.git device/xiaomi/ginkgo
git clone -b lineage-19.1 https://github.com/LineageOS/android_device_xiaomi_sm6125-common.git device/xiaomi/sm6125-common
```
Утилиту для конвертирования dat в img:
```sh
git clone https://github.com/xpirt/sdat2img
```

Если в качестве основы используется crDroid, то момманды будут следующие:
```sh
git clone https://github.com/crdroidandroid/android_tools_extract-utils.git tools/extract-utils
git clone -b 13.0 https://github.com/crdroidandroid/android_device_xiaomi_ginkgo.git device/xiaomi/ginkgo
```
После этого можно заняться распаковкой образов system и vendor из скачанного ранее архива с прошивкой.
Для этого копируем/переносим архив этот же каталог (~/AndroidBuild/ginkgo_dump) и распаковываем его:
```sh
unzip miui_GINKGOEEAGlobal_V12.5.12.0.RCOEUXM_f9fdc6c783_11.0.zip
```
Нас интересуют только файлы: `system.new.dat.br`, `system.transfer.list`, `vendor.new.dat.br`, `vendor.transfer.list`.
Файлы `*.transfer.list` понадобятся для распаковки split-файлов.

Файлы `*.dat.br` - это архивы, созданные с помощью утилиты `brotli`. Для распаковки таких файлов нужно установить саму утилиту:
```sh
sudo apt install brotli
```
и выполнить:
```sh
brotli --decompress --output=system.new.dat system.new.dat.br
brotli --decompress --output=vendor.new.dat vendor.new.dat.br
```
Так мы получим `system.new.dat` и `vendor.new.dat`.

Теперь нужно извлечь из файлов `*.dat` (это те самые split-архивы) соответствующие образы `*.img`.
Для этого выполняем:
```sh
python sdat2img/sdat2img.py system.transfer.list system.new.dat system.img
python sdat2img/sdat2img.py vendor.transfer.list vendor.new.dat vendor.img
```
Удаляем всё, что не нужно:
```sh
rm -rf META-INF firmware-update boot.img *.patch.dat *.zip system.*.* vendor.*.*
```
Монтируем полученные образы в дерево файловой системы:
```sh
mkdir system
sudo mount system.img system
sudo mount vendor.img system/vendor
```
Тут создаётся каталог `system`, в него монтируется `system.img`, и после этого уже в имеющуюся в `system` папку `vendor` монтируется `vendor.img`.

Можно приступать к извлечению файлов. 
Выполняем:
```sh
cd device/xiaomi/ginkgo/
./extract-files.sh ../../../
```
После этого в каталоге `~/AndroidBuild/ginkgo_dump/vendor` появятся извлечённые из образов файлы.

Отмонтируем ненужные теперь образы:
```sh
sudo umount system/vendor
sudo umount system
```
и удаляем всё, что нам теперь не нужно:
```sh
cd ../../../
rm -rf *.img sdat2img tools device system
```
В итоге останется только каталог `vendor` с извлечёнными из образов файлами.



