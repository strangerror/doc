# Сборка Android из исходников

## Сборка
В качестве примера прошивки используется [crDroid](https://github.com/crdroidandroid), исходники для устройства от [garry-rogov](https://github.com/garry-rogov).<br>
Ситема собирается для устройства Redmi Note 8/8T.

Сборка выполняется в `Ubuntu` или любой другой системе, производной от неё.<br>
Первоначально нужно установить пакеты, указанные [здесь](https://source.android.com/docs/setup/start/initializing#installing-required-packages-ubuntu-1804):
```sh
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev libc6-dev-i386 libncurses5 x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig
```
Зачастую, команды разработчиков кастомных прошивок дополнительно указвают в манифесте для своих систем, какие пакеты нужно уставновить. Иногда там есть те, которых нет в руководстве от `Google`. Их желательно тоже установить.
Для `crDroid` перечень пактов указан [здесь](https://github.com/crdroidandroid/android#11-installing-dependencies-and-repo):
```sh
sudo apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git git-lfs gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libwxgtk3.0-gtk3-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev
```
Нужно создать структуру каталогов. В качестве примера указано, как это сделано у меня.<br>
В домашнем каталоге создаём каталог, в котором будет храниться всё, что связано со сборкой:
```sh
mkdir ~/AndroidBuild
```
Создаём каталог, в котором будут находиться исходники системы - `crDroid`.
```sh
mkdir ~/AndroidBuild/crDroid
```
Для скачивания исходников используется утилита [`repo`](https://gerrit.googlesource.com/git-repo). Установим её:
```sh
mkdir ~/AndroidBuild/bin && curl https://storage.googleapis.com/git-repo-downloads/repo > ~/AndroidBuild/bin/repo && chmod a+x ~/AndroidBuild/bin/repo
```
Здесь создаётся подкаталог `bin`, в который скачивается `repo`. На `repo` устанавливаются права, позволяющие запускать его как исполняемый файл.<br>
Так же, можно установить с помощью пакетного менеджера, если нужный пакет есть в репозиториях:
```sh
sudo apt-get install repo
```
Теперь нужно добавить каталог с `repo` в переменные окружения. Что бы это происходило при загрузке системы нужно отредактировать файл `.bashrc`, находящийся в каталоге профиля.
В самом конце файла нужно добавить строку:
```sh
export PATH=~/AndroidBuild/bin:$PATH
```
Что бы применить изменения сразу, а не после перезагрузки нужно выполнить в терминале:
```sh
source ~/.bashrc
```

Теперь можно инициализировать локальный репозиторий и скачивать исходники. Для этого в терминале выполнить:
```sh
cd ~/AndroidBuild/crDroid
git config --global user.email "e-mail"
git config --global user.name "nickname"
repo init -u https://github.com/crdroidandroid/android.git -b 13.0 --git-lfs
```
Здесь переходим в каталог для хранения исходнииков системы, созданный ранее. Указываем, как `git` будет идентифицировать нас - нужно указать адрес электронной почты и псевдоним. Инициализируем репозиторий в каталоге, в котором находимся.<br>
Команда для инициализации обычно указана в манифесте системы, которую мы собираем. Для `crDroid` она указана [здесь](https://github.com/crdroidandroid/android#12-initializing-repo).

После этого, находясь в каталоге, в котором инициализировали репозиторий, запускаем скачивание исходников системы. В общем виде команда выглядит так:
```sh
repo sync
```
Я использую:
```sh
repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j$(nproc --all)
```
Рекомендованная авторами прошивки команда указана, опять же, в манифесте системы.<br>
Отступление от рекомендаций может привести к проблемам во время сборки. Поэтому, если проблемы возникают, стоит выполнить:
```sh
repo sync
```

После скачивания исходников системы к ним нужно добавить ещё исходники для конкретного устройства. Для этого в каталоге с исходниками ищем подкаталог `.repo` - он скрытый.
В нём нужно создать подкаталог `local_manifests` и положить в него xml-файлы, указывающие, откуда качать исходники для устройства. Имена и количество xml-файлов значения не имеют. Важно содержимое.<br>
Т.к. я использую исходники от `garry-rogov`, то у меня содержимое файла выглядит так:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <remote name="garry-rogov"
          fetch="https://github.com/garry-rogov" />  
  <!-- Kernel Tree -->
  <project name="android_kernel_xiaomi_ginkgo" path="kernel/xiaomi/ginkgo" remote="garry-rogov" revision="13" />  
  <!-- Device Tree -->
  <project name="android_device_xiaomi_willow" path="device/xiaomi/willow" remote="garry-rogov" revision="13" />
  <!-- Vendor Tree -->
  <project name="android_vendor_xiaomi_ginkgo" path="vendor/xiaomi/ginkgo" remote="garry-rogov" revision="13" />  
</manifest>
```
Если есть желание добавить камеру MIUI, то нужен ещё один файл со следующим содержимым:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <remote name="garry-rogov"
          fetch="https://github.com/garry-rogov" />
  <!-- MiuiCamera --> 
    <project name="android_vendor_miuicamera" path="vendor/miuicamera" remote="garry-rogov" revision="arrow-12.0-a3" />
</manifest>
``` 
Аналогично можно добавить в прошивку любое ПО, главное найти нужный репозиторий.
Теперь нужно ещё раз выполнить синхронизацию:
```sh
repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j$(nproc --all)
```
Можно запускать сборку. Для этого в каталоге с исходниками нужно выполнить:
```sh
. build/envsetup.sh
brunch willow
```
Если нужно ограничить количество потоков при сборке (в примере - 4), то выполнять:
```sh
. build/envsetup.sh
lunch
make bacon -j4
```
Очень помогает при сборке использование `ccache`. Для его использования перед началом сборки нужно экспортировать несколько параметров:
```sh
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
export CCACHE_DIR=~/AndroidBuild/.ccache/crDroid
ccache -M 50G -F 0
```
Здесь мы указываем, что нужно использовать `ccache`, где находится его бинарник (обычно этого не требуется и система сама знает, где он находится), где хранится кэш для сборки и размер кэша (эти данные достаточно задать один раз и потом они будут браться из конфиг-файла в каталоге с кэшем).

После окончания сборки файл прошивки будет находится в подкаталоге `out` каталога с исходниками собираемой системы.
Переопределить его расположение можно самостоятельно, задав значение параметра `OUT_DIR`. Для этого нужно выполнить:
```sh
export OUT_DIR="путь_к_каталогу_out"
```
  
## Возможные проблемы при сборке
1. Если при инициализации репозитория выдаётся ошибка:
   ```sh
   /usr/bin/env: 'python': No such file or directory
   ```
   то можно сделать следующее:
   * сделать симлинк на /usr/bin/python
     ```sh
     ln -s /usr/bin/python3 /usr/bin/python
     ```
   * или, что лучше
     ```sh
     sudo apt install python-is-python3
     ```
   Информация об этом есть в [офф руководстве](https://source.android.com/docs/setup/download/downloading) и на [stackoverflow](https://stackoverflow.com/questions/3655306/ubuntu-usr-bin-env-python-no-such-file-or-directory).

2. Сборка падает со следующей ошибкой:
   ```sh
   FAILED: out/soong/build.ninja
   ......
   soong bootstrap failed with: exit status 1
   ```
   Возникает скорее всего из-за нехватки оперативной памяти. Где-то в интернете встречалась информация, что на один поток требуется 2.3 - 2.5 Гб RAM.<br>
   Нужно попробовать добавить файл подкачки. Для этого выполнить:
   ```sh
   sudo dd if=/dev/zero of=/swapfile bs=1G count=70 status=progress
   sudo chmod 0600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile
   ```
   Здесь создаём файл размером 70 Гб в корне системы, устанавливаем нужные права на него, "форматируем" его и подключаем.
   Проверить размер файла подкачки можно командой:
   ```sh
   sudo swapon --show
   ```
   Если нужно, что бы файл подкачки подключался при загрузке системы, то следует добавить в файл `fstab` строку:
   ```sh
   /swapfile   none  swap  defaults  0  0
   ```

3. Ошибка при компиляции `webview.apk`:
   ```sh
   external/chromium-webview/prebuilt/arm64/webview.apk: Invalid file ERROR: dump failed because no AndroidManifest.xml found
   ```
   Решение - удалить всё, что связано с `webview` из исходников и выполнить повторно синхронизацию:
   ```sh
   rm -rf external/chromium-webview/prebuilt/*
   rm -rf .repo/projects/external/chromium-webview/prebuilt/*.git
   rm -rf .repo/project-objects/LineageOS/android_external_chromium-webview_prebuilt_*.git
   repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j$(nproc --all)
   ```
   Решение взято с [reddit](https://www.reddit.com/r/LineageOS/comments/11rnaoi/webview_wont_compile_after_the_recent_git_lfs/).<br>
   Так же можно попробовать [установить](https://docs.github.com/en/repositories/working-with-files/managing-large-files/installing-git-large-file-storage) (если не был установлен ранее) `git-lfs` перед выполнением команд, указанных выше.

4. При выполнении `repo sync` получаем сообщение:
   ```sh
   Ваши локальные изменения в указанных файлах будут перезаписаны при переключении на состояние...
   Сделайте коммит или спрячьте ваши изменения перед переключением веток.
   ```
   Решение - перейти в каталог репозитория, с синхронизацией которого возникла проблема (указывается в тексте ошибки), "спрятать" свои изменения, выполнить синхронизацию и веруть изменения обратно:
   ```sh
   cd packages/apps/crDroidSettings
   git stash
   repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j$(nproc --all)
   git stash pop
   ```
   Поздробнее можно почитать [здесь](https://www.atlassian.com/ru/git/tutorials/saving-changes/git-stash), [здесь](https://selectel.ru/blog/tutorials/git-stash-manual/) или [здесь](https://git-scm.com/book/en/v2/Git-Tools-Stashing-and-Cleaning).

5. Ошибка при сборке sepolicy:
   ```sh
   FAILED: out/soong/.intermediates/system/sepolicy/plat_sepolicy.cil/android_common/plat_sepolicy.cil
   out/host/linux-x86/bin/checkpolicy -C -M -c 30 -o out/soong/.intermediates/system/sepolicy/plat_sepolicy.cil/android_common/plat_sepolicy.cil out/soong/.intermediates/system/sepolicy/plat_sepolicy.conf/android_common/plat_sepolicy.conf && cat system/sepolicy/private/technical_debt.cil >>  out/soong/.intermediates/system/sepolicy/plat_sepolicy.cil/android_common/plat_sepolicy.cil && out/host/linux-x86/bin/secilc -m -M true -G -c 30 out/soong/.intermediates/system/sepolicy/plat_sepolicy.cil/android_common/plat_sepolicy.cil -o /dev/null -f /dev/null # hash of input list: adbae54a02efb1ac28c9b9ece27d8ae5a4fc7adea217e7e24e3ecb8996f196b7
   neverallow check failed at out/soong/.intermediates/system/sepolicy/plat_sepolicy.cil/android_common/plat_sepolicy.cil:20503 from system/sepolicy/private/domain.te:408
     (neverallow base_typeattr_687 self (cap_userns (dac_read_search)))
       <root>
       allow at out/soong/.intermediates/system/sepolicy/plat_sepolicy.cil/android_common/plat_sepolicy.cil:22935
         (allow logpersist self (cap_userns (dac_override dac_read_search sys_nice)))

   neverallow check failed at out/soong/.intermediates/system/sepolicy/plat_sepolicy.cil/android_common/plat_sepolicy.cil:20502 from system/sepolicy/private/domain.te:408
     (neverallow base_typeattr_687 self (capability (dac_read_search)))
   ...
   Failed to generate binary
   Failed to build policydb
   ```
   В качестве <font color="red">**временного**</font> решения можно добавить `SELINUX_IGNORE_NEVERALLOWS := true` в `BoardConfig.mk`.