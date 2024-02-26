---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhrynova
tested on: Всі версії
tags:
  - моніторинг системи
  - моніторинг процесу
  - fuser
  - numactl
  - perf
  - chrt
  - schedtool
  - atop
  - strace
  - systemd-run
  - taskset
  - cgroups
---

# Лабораторна робота 4: Розширений моніторинг системи та процесів

## Завдання

Виконавши цю лабораторну роботу, ви зможете

- переглядати та керувати процесами за допомогою передових інструментів
- діагностувати та налагоджувати системні виклики
- переглядати та встановлювати пріоритет процесу за допомогою розширених інструментів CLI
- переглядати та встановлювати власні політики планування для процесів
- аналізувати продуктивність системи та додатків

Приблизний час виконання цієї лабораторної роботи: 90 хвилин

## Вступ

Команди в цій лабораторній роботі охоплюють ширший спектр керування процесами, моніторингу системи та керування ресурсами в Linux. Вони додають більшої глибини та різноманітності вашому репертуару системного адміністратора.

Ці вправи охоплюють додаткові команди та поняття Linux, надаючи практичний досвід керування процесами, моніторингу ресурсів і розширеного контролю.

## Завдання 1

### fuser

Команда `fuser` в Linux використовується для ідентифікації процесів за допомогою файлів або сокетів. Це може бути корисною підмогою в управлінні процесами, пов’язаними з файлами, і вирішенні конфліктів.

#### Щоб створити сценарій для імітації використання файлу

1. Спочатку створіть порожній тестовий файл, до якого ми хочемо отримати доступ. Впишіть:

   ```bash
   touch ~/testfile.txt
   ```

2. Створіть сценарій, який ми будемо використовувати для імітації доступу до testfile.txt. Впишіть:

   ```bash
   cat > ~/simulate_file_usage.sh << EOF
   #!/bin/bash
   tail -f ~/testfile.txt
   EOF   
   ```

3. Зробіть сценарій виконуваним. Впишіть:

   ```bash
   chmod +x ~/simulate_file_usage.sh
   ```

4. Запустіть скрипт. Впишіть:

   ```bash
   ~/simulate_file_usage.sh &
   ```

#### Щоб ідентифікувати процеси, які звертаються до файлу

1. Визначте процеси за допомогою або доступу до `testfile.txt`, запустіть:

   ```bash
   fuser ~/testfile.txt
   ```

2. Перегляньте додаткові параметри `fuser` за допомогою параметра `-v`. Впишіть:

   ```bash
   fuser -v ~/testfile.txt
   ```

3. Перегляньте додаткові параметри `fuser` за допомогою параметра `-v`. Тепер ви можете видалити файли. Впишіть:

   ```bash
   kill %1
   rm ~/testfile.txt ~/simulate_file_usage.sh
   ```

#### Щоб ідентифікувати процес, який отримує доступ до порту TCP/UDP

1. Використовуйте команду `fuser`, щоб визначити процес доступу до TCP-порту 22 на вашому сервері. Впишіть:

   ```bash
   sudo fuser 22/tcp
   ```

## Завдання 2

### `perf`

`perf` — це універсальний інструмент для аналізу продуктивності системи та програм у Linux. Він може запропонувати додаткові відомості, які можуть допомогти налаштувати продуктивність.

#### Щоб встановити `perf`

1. Встановіть програму `perf`, якщо вона не встановлена на вашому сервері. Впишіть:

   ```bash
   sudo dnf -y install perf
   ```

2. Програма `bc` — це точний калькулятор командного рядка. `bc` буде використовуватися в цій вправі для імітації високого навантаження ЦП. Якщо `bc` ще не встановлено на вашому сервері, встановіть його за допомогою:

   ```bash
   sudo dnf -y install bc
   ```

#### Щоб створити сценарій для генерації навантаження на процесор

1. Створіть сценарій завантаження ЦП і зробіть його виконуваним, виконавши:

   ```bash
   cat > ~/generate_cpu_load.sh << EOF
   #!/bin/bash

   # Check if the number of decimal places is passed as an argument
   if [ "$#" -ne 1 ]; then
     echo "Usage: $0 <number_of_decimal_places>"
     exit 1
   fi

   # Calculate Pi to the specified number of decimal places
   for i in {1..10}; do echo "scale=$1; 4*a(1)" | bc -l; done

   EOF
   chmod +x ~/generate_cpu_load.sh
   ```

   !!! tip "Підказка"

   ```
    Сценарій generate_cpu_load.sh — це простий інструмент для генерування навантаження на ЦП шляхом обчислення Пі (π) з високою точністю. Такий же розрахунок проводиться десять разів. Сценарій приймає ціле число як параметр для визначення кількості знаків після коми для обчислення Пі.
   ```

#### Для імітації додаткового навантаження на процесор

1. Давайте проведемо простий тест і обчислимо Пі з точністю до 50 знаків після коми. Запустіть сценарій, ввівши:

   ```bash
   ~/generate_cpu_load.sh 50 & 
   ```

2. Повторно запустіть сценарій, але використовуйте `perf`, щоб записати продуктивність сценарію для аналізу використання ЦП та інших показників. Впишіть:

   ```bash

    ./generate_cpu_load.sh 1000  &  perf record -p $! sleep 5
   ```

   !!! tip "Підказка"

   ```
    Параметр `sleep 5` з командою `perf record` визначає часове вікно для `perf` для збору даних про продуктивність щодо навантаження ЦП, створених сценарієм generate_cpu_load.sh. Це дозволяє `perf записувати показники продуктивності системи протягом 5 секунд перед автоматичною зупинкою.
   ```

#### Для аналізу даних про ефективність і моніторингу подій у реальному часі

1. Використовуйте команду `perf report`, щоб переглянути звіт про продуктивність, щоб зрозуміти шаблони використання процесора та пам’яті. Впишіть:

   ```bash
   sudo perf report
   ```

   Ви можете використовувати різні клавіші клавіатури для подальшого вивчення звіту.
   Введіть ++"q"++, щоб вийти з інтерфейсу перегляду звітів `perf`.

2. Спостерігайте/фіксуйте події кешу процесора в режимі реального часу протягом 40 секунд, щоб виявити потенційні вузькі місця продуктивності. Впишіть:

   ```bash
   sudo perf stat -e cache-references,cache-misses sleep 40
   ```

#### Щоб записати повну продуктивність системи

1. Збирайте дані про продуктивність усієї системи, які можна використовувати для додаткового аналізу. Впишіть:

   ```bash
   sudo perf record -a sleep 10
   ```

2. Досліджуйте лічильники конкретних подій. Підраховуйте конкретні події, як-от цикли процесора, щоб оцінити продуктивність певного сценарію чи програми. Давайте перевіримо за допомогою простої команди `find`, введіть:

   ```bash
   sudo perf stat -e cycles find /proc
   ```

3. Зробіть те саме, але зі сценарієм ./generate_cpu_load.sh. Підраховуйте певні події, як-от цикли ЦП, щоб оцінити продуктивність сценарію ./generate_cpu_load.sh. Впишіть:

   ```bash
   sudo perf stat -e cycles ./generate_cpu_load.sh 500
   ```

   Вихід

   ```bash
   ...<SNIP>...
   3.141592653589793238462643383279502884197169399375105820974944592307\
   81640628620899862803482534211.....

   Performance counter stats for './generate_cpu_load.sh 500':

     1,670,638,886      cycles

        0.530479014 seconds time elapsed

        0.488580000 seconds user
        0.034628000 seconds sys
   ```

   !!! note "Примітка"

   ```
    Ось розбивка кінцевого зразка результату команди `perf stat`:
    
    *1,670,638,886 cycles*: Це вказує на загальну кількість циклів процесора, споживаних під час виконання сценарію. Кожен цикл являє собою один крок у виконанні інструкцій ЦП.

    *0.530479014 seconds time elapsed*: Це загальний реальний час (або час настінного годинника), що минув від початку до кінця виконання сценарію. Ця тривалість включає всі типи очікувань (наприклад, очікування дискового вводу-виводу або системних викликів).

    *0.488580000 seconds user*: Це час роботи процесора в режимі користувача. Цей час явно виключає час, витрачений на виконання завдань системного рівня.  

    *0.034628000 seconds sys*: Це час роботи процесора в режимі ядра або системи. Це включає час, який ЦП витрачає на виконання системних викликів або виконання інших завдань системного рівня від імені сценарію.
   ```

4. Усе зроблено за допомогою інструмента `perf`. Переконайтеся, що фонові сценарії призначені для чистого робочого середовища.

   ```bash
   kill %1
   ```

## Завдання 3

### `strace`

`strace` використовується для діагностики та налагодження взаємодії системних викликів у Linux.

#### Щоб створити сценарій для дослідження `strace`

1. Створіть простий сценарій під назвою `strace_script.sh` і зробіть його виконуваним. Впишіть:

   ```bash
   cat > ~/strace_script.sh << EOF
   #!/bin/bash
   while true; do
     date
     sleep 1
   done
   EOF
   chmod +x ~/strace_script.sh
   ```

#### Щоб використовувати `strace` для запущених процесів

1. Запустіть сценарій і прикріпіть `strace`. Впишіть:

   ```bash
   ~/strace_script.sh &
   ```

2. Знайдіть PID для процесу `strace_script.sh` в окремому терміналі. Зберігайте PID у змінній з іменем MYPID. Для цього ми використаємо команду `pgrep`, виконавши:

   ```bash
   export MYPID=$(pgrep strace_script) ; echo $MYPID
   ```

   Вихід

   ```bash
   4006301
   ```

3. Почніть відстежувати системні виклики сценарію, щоб зрозуміти, як він взаємодіє з ядром. Приєднайте `strace` до запущеного процесу сценарію, ввівши:

   ```bash
   sudo strace -p $MYPID
   ```

4. Від’єднайте або зупиніть процес `strace`, ввівши ++ctrl+c++

5. Вихід `strace` можна відфільтрувати, зосередившись на певних системних викликах, таких як `open` і `read`, щоб проаналізувати їх поведінку. Спробуйте зробити це для системних викликів `open` і `read`. Впишіть:

   ```bash
   sudo strace -e trace=open,read -p $MYPID
   ```

   Коли ви завершите спробу розшифрувати вихідні дані `strace`, зупиніть процес `strace`, ввівши ++ctrl+c++

6. Перенаправте вихідні дані у файл для подальшого аналізу, який може допомогти діагностувати проблеми. Збережіть вивід `strace` у файл, виконавши:

   ```bash
   sudo strace -o strace_output.txt -p $MYPID
   ```

#### Для аналізу частоти системних викликів

1. Узагальніть кількість системних викликів, щоб визначити системні виклики, які найчастіше використовуються процесом. Зробіть це лише на 10 секунд, додавши команду `timeout`. Впишіть:

   ```bash
   sudo timeout 10 strace -c -p $MYPID
   ```

   Наша зразкова система показує підсумковий звіт таким чином:

   Вихід

   ```bash
   strace: Process 4006301 attached
   strace: Process 4006301 detached
   % time     seconds  usecs/call     calls    errors syscall
   ------ ----------- ----------- --------- --------- ----------------
   89.59    0.042553        1182        36        18 wait4
   7.68    0.003648         202        18           clone
   1.67    0.000794           5       144           rt_sigprocmask
   0.45    0.000215           5        36           rt_sigaction
   0.36    0.000169           9        18           ioctl
   0.25    0.000119           6        18           rt_sigreturn
   ------ ----------- ----------- --------- --------- ----------------
   100.00    0.047498         175       270        18 total
   ```

2. Припиніть виконання сценарію та видаліть усі створені файли.

   ```bash
   kill $MYPID
   rm ~/strace_script.sh ~/strace_output.txt
   ```

## Завдання 4

### `atop`

`atop` надає повне уявлення про продуктивність системи, охоплюючи різні показники ресурсів.

#### Щоб запустити та дослідити `atop`

1. Встановіть програму `atop`, якщо вона не встановлена на вашому сервері. Впишіть:

   ```bash
   sudo dnf -y install atop
   ```

2. Запустіть `atop`, ввівши:

   ```bash
   sudo atop
   ```

3. В інтерфейсі `atop` ви можете досліджувати різні показники `atop`, натискаючи певні клавіші на клавіатурі.

   Використовуйте 'm', 'd' або 'n' для перемикання між переглядами пам'яті, диска або мережі. Спостерігайте, як використовуються ресурси в реальному часі.

4. Відстежуйте продуктивність системи з користувацьким інтервалом у 2 секунди, що дозволяє більш детально переглядати діяльність системи. Впишіть:

   ```bash
   sudo atop 2
   ```

5. Перемикайтеся між різними видами ресурсів, щоб зосередитися на конкретних аспектах продуктивності системи.

6. Створіть звіт із файлу журналу про діяльність системи, збираючи дані кожні 60 секунд, тричі. Впишіть:

   ```bash
   sudo atop -w /tmp/atop_log 60 3
   ```

7. Після виконання попередньої команди ви можете не поспішати та переглянути двійковий файл, у якому було збережено журнали. Щоб прочитати збережений файл журналу, введіть:

   ```bash
   sudo atop -r /tmp/atop_log   
   ```

8. Очистіть, видаливши всі створені журнали або файли.

   ```bash
   sudo rm /tmp/atop_log
   ```

## Завдання 5

### `numactl`

NUMA означає "нерівномірний доступ до пам'яті".
Це конструкція/архітектура комп’ютерної пам’яті, що використовується в багатопроцесорній обробці, яка підвищує швидкість доступу до пам’яті, враховуючи фізичне розташування пам’яті навколо процесорів.

Програма `numactl` керує політикою NUMA, оптимізуючи продуктивність систем на основі NUMA.

#### Щоб встановити `numactl`

1. Встановіть програму `numactl`, якщо вона не встановлена на вашому сервері. Впишіть:

   ```bash
   sudo dnf -y install numactl
   ```

#### Для створення сценарію, що потребує великої кількості пам’яті

1. Створіть простий сценарій, який допоможе імітувати робоче навантаження з інтенсивним використанням пам’яті на вашому сервері. Впишіть:

   ```bash
   cat > ~/memory_intensive.sh << EOF
     #!/bin/bash

     awk 'BEGIN{for(i=0;i<1000000;i++)for(j=0;j<1000;j++);}{}'
     EOF
     chmod +x ~/memory_intensive.sh
   ```

#### Щоб використовувати `numactl`

1. Запустіть сценарій за допомогою `numactl`, введіть:

   ```bash
   numactl --membind=0 ~/memory_intensive.sh
   ```

2. Якщо ваша система має більше ніж один доступний вузол NUMA, ви можете запустити сценарій на кількох вузлах NUMA за допомогою:

   ```bash
   numactl --cpunodebind=0,1 --membind=0,1 ~/memory_intensive.sh
   ```

3. Покажіть розподіл пам'яті на вузлах NUMA

   ```bash
   numactl --show
   ```

4. Прив’яжіть пам’ять до певного вузла, виконавши:

   ```bash
   numactl --membind=0 ~/memory_intensive.sh
   ```

5. Очистіть своє робоче середовище, видаливши сценарій.

   ```bash
   rm ~/memory_intensive.sh
   ```

## Завдання 6

### `iotop`

Команда `iotop` відстежує використання дискового введення/виведення (введення/виведення) процесами та потоками. Він надає інформацію в реальному часі, подібну до команди `top`, зокрема для дискового вводу-виводу. Це робить його необхідним для діагностики уповільнення роботи системи, викликаного активністю диска.

#### Щоб встановити `iotop`

1. Встановіть утиліту `iotop`, якщо вона не встановлена. Впишіть:

   ```bash
   sudo dnf -y install iotop
   ```

#### Щоб використовувати `iotop` для моніторингу дискового введення-виведення

1. Запустіть команду `iotop` без будь-яких параметрів, щоб використовувати її в інтерактивному режимі за замовчуванням. Впишіть:

   ```bash
   sudo iotop
   ```

   Спостерігайте за використанням живого диска вводу-виводу різними процесами. Використовуйте це для визначення процесів, які зараз читають або записують на диск.

2. Введіть ++"q"++, щоб вийти з `iotop`.

#### Для використання `iotop` в неінтерактивному режимі

1. Запустіть `iotop` у пакетному режимі (-b), щоб отримати неінтерактивний одноразовий перегляд використання введення-виведення. Параметр `-n 10` повідомляє `iotop` взяти 10 зразків перед виходом.

   ```bash
   sudo iotop -b -n 10
   ```

2. `iotop` може фільтрувати введення/виведення для певних процесів. Визначте ідентифікатор процесу (PID) у вашій системі за допомогою команди ps або дисплея `iotop`. Потім відфільтруйте вихідні дані `iotop` для цього конкретного PID. Наприклад, фільтруйте PID для процесу `sshd`, виконавши:

   ```bash
   sudo iotop -p $(pgrep sshd | head -1)
   ```

3. Опцію -`o` з `iotop` можна використовувати для показу процесів або потоків, які виконують фактичний ввід-вивід, замість відображення всіх процесів або потоків. Відобразити лише процеси вводу-виводу, виконавши:

   ```bash
   sudo iotop -o
   ```

   !!! Question "Обговорення"

   ```
    Обговоріть вплив дискового вводу-виводу на загальну продуктивність системи та як такі інструменти, як `iotop`, можуть допомогти в оптимізації системи.
   ```

## Завдання 7

### `cgroups`

Групи керування (`cgroups`) забезпечують механізм у Linux для організації, обмеження та визначення пріоритетів використання ресурсів процесами.

Ця вправа демонструє пряму взаємодію з файловою системою `cgroup` v2.

### Для вивчення файлової системи `cgroup`

1. Використовуйте команду `ls`, щоб дослідити вміст і структуру файлової системи `cgroup`. Впишіть:

   ```bash
   ls /sys/fs/cgroup/
   ```

2. Знову скористайтеся командою `ls`, щоб отримати список папок \*.slice у файловій системі `cgroup`. Впишіть:

   ```bash
   ls -d /sys/fs/cgroup/*.slice
   ```

   Папки з суфіксом .slice зазвичай використовуються в `systemd` для представлення фрагмента системних ресурсів. Це стандартні `cgroups`, якими керує `systemd` для організації та керування системними процесами.

### Щоб створити власну `cgroup`

1. Створіть каталог під назвою "exercise_group" у файловій системі /sys/fs/cgroup. У цій новій папці будуть розміщені структури контрольних груп, необхідні для решти цієї вправи. Використовуйте команду `mkdir`, ввівши:

   ```bash
   sudo mkdir -p /sys/fs/cgroup/exercise_group
   ```

2. Тепер перелічіть файли та каталоги в структурі /sys/fs/cgroup/exercise_group. Впишіть:

   ```bash
   sudo ls /sys/fs/cgroup/exercise_group/
   ```

   Вихідні дані показують файли та каталоги, автоматично створені підсистемою `cgroup` для керування та моніторингу ресурсів для `cgroup`.

#### Щоб встановити новий ліміт ресурсу пам'яті

1. Давайте встановимо обмеження ресурсу пам’яті, щоб обмежити використання пам’яті до 4096 байт (4 КБ). Щоб обмежити процеси в `cgroup` використовувати максимум 4 Кб пам'яті типу:

   ```bash
   echo 4096 | sudo tee /sys/fs/cgroup/exercise_group/memory.max
   ```

2. Підтвердьте, що ліміт пам’яті встановлено. Впишіть:

   ```bash
   cat /sys/fs/cgroup/exercise_group/memory.max
   ```

#### Щоб створити тестовий сценарій memory_stress

1. Створіть простий виконуваний сценарій за допомогою команди `dd`, щоб перевірити обмеження ресурсу пам’яті. Впишіть:

   ```bash
   cat > ~/memory_stress.sh << EOF
   #!/bin/bash
   dd if=/dev/zero of=/tmp/stress_test bs=10M count=2000
   EOF
   chmod +x ~/memory_stress.sh
   ```

#### Щоб запустити та додати процес/сценарій до `cgroup` пам'яті

1. Запустіть memory_stress.sh, запишіть його PID і додайте PID до group.procs. Впишіть:

   ```bash
   ~/memory_stress.sh &
   echo $! | sudo tee /sys/fs/cgroup/exercise_group/cgroup.procs
   ```

   Файл /sys/fs/cgroup/exercise_group/cgroup.procs можна використовувати для додавання або перегляду PID (ідентифікаторів процесів) процесів, які є членами даної `cgroup`. Запис PID до цього файлу призначає процес сценарію \~/memory_stress.sh групі `cgroup`.

2. Попередня команда завершиться дуже швидко, тому що вона перевищила обмеження пам’яті `cgroup`. Ви можете запустити таку команду `journalctl` в іншому терміналі, щоб переглянути помилку, як вона відбувається. Впишіть:

   ```bash
   journalctl -xe -f  | grep -i memory
   ```

   !!! tip "Порада"

   ````
    Ви можете швидко скористатися командою ps, щоб перевірити приблизне використання пам'яті процесом, якщо вам відомий PID процесу, запустивши:

    ```bash
    pidof <PROCESS_NAME> | xargs ps -o pid,comm,rss
    ```

    Цей вихід має відображати розмір резидентного набору (RSS) у КБ, що вказує на пам’ять, яку використовує вказаний процес у певний момент часу. Кожного разу, коли значення RSS процесу перевищує ліміт пам’яті, вказаний у значенні memory.max `cgroup's`, процес може підлягати політикам керування пам’яттю, які застосовуються ядром або самою `cgroup`. Залежно від конфігурації системи, система може виконувати такі дії, як обмеження використання пам’яті процесом, припинення процесу або ініціювання події браку пам’яті (OOM).
   ````

#### Щоб установити новий ліміт ресурсу ЦП

1. Обмежте використання сценарієм лише 10% ядра ЦП. Впишіть:

   ```bash
   echo 10000 | sudo tee /sys/fs/cgroup/exercise_group/cpu.max
   ```

   10000 означає обмеження пропускної здатності ЦП. Його встановлено на 10% від загальної потужності одного ядра ЦП.

2. Переконайтеся, що встановлено обмеження ЦП. Впишіть:

   ```bash
   cat /sys/fs/cgroup/exercise_group/cpu.max
   ```

#### Щоб створити сценарій стрес-тесту ЦП

1. Створюйте та встановлюйте дозволи на виконання для сценарію, щоб генерувати високе використання ЦП. Впишіть:

   ```bash
   cat > ~/cpu_stress.sh << EOF
   #!/bin/bash
   exec yes > /dev/null
   EOF
   chmod +x ~/cpu_stress.sh
   ```

   !!! note "Примітка"

   ```
     `yes > /dev/null` — це проста команда, яка створює високе навантаження на ЦП.
   ```

#### Щоб запустити та додати процес/сценарій до CPU `cgroup`

1. Запустіть сценарій і негайно додайте його PID до `cgroup`, ввівши:

   ```bash
   ~/cpu_stress.sh &
   echo $! | sudo tee /sys/fs/cgroup/exercise_group/cgroup.procs
   ```

#### Щоб підтвердити контроль використання ЦП процесу

1. Перевірте використання процесора процесом.

   ```bash
   pidof yes | xargs top -b -n 1 -p
   ```

   Вихідні дані мають показувати використання ЦП у реальному часі процесом yes. %CPU для так має бути обмежено конфігурацією `cgroup` (наприклад, близько 10%, якщо обмеження встановлено на 10000).

2. Встановіть та поекспериментуйте з іншими значеннями для cpu.max для `cgroup` групи виконання, а потім спостерігайте за ефектом кожного разу, коли ви повторно запускаєте сценарій \~/cpu_stress.sh у контрольній групі.

#### Щоб визначити та вибрати основний запам'ятовуючий пристрій

Основний запам'ятовуючий пристрій може бути ціллю для встановлення обмежень на ресурси введення-виведення. Пристрої зберігання даних у системах Linux мають основні та другорядні номери пристроїв, які можна використовувати для їх унікальної ідентифікації.

1. По-перше, давайте створимо набір допоміжних змінних для визначення та збереження номера пристрою для основного пристрою зберігання на сервері. Впишіть:

   ```bash
   primary_device=$(lsblk | grep disk | awk '{print $1}' | head -n 1)
   primary_device_num=$(ls -l /dev/$primary_device | awk '{print $5, $6}' | sed 's/,/:/')
   ```

2. Далі, відобразимо значення змінної $primary_device_num. Впишіть:

   ```bash
   echo "Primary Storage Device Number: $primary_device_num"
   ```

3. Головний і допоміжний номери пристроїв мають відповідати тому, що ви бачите в цьому виведеному файлі ls:

   ```bash
     ls -l /dev/$primary_device
   ```

#### Щоб установити нове обмеження ресурсів введення-виведення

1. Встановіть операції вводу/виводу на 1 МБ/с для процесів читання та запису в `cgroup` групи виконання. Впишіть:

   ```bash
   echo "$primary_device_num rbps=1048576 wbps=1048576" | \
   sudo tee /sys/fs/cgroup/exercise_group/io.max
   ```

2. Підтвердьте встановлені обмеження введення/виведення. Впишіть:

   ```bash
   cat /sys/fs/cgroup/exercise_group/io.max
   ```

#### Щоб створити процес стрес-тесту вводу-виводу

1. Запустіть процес `dd`, щоб створити великий файл з назвою /tmp/io_stress. Також запишіть і збережіть PID процесу dd у змінній під назвою `MYPID`. Впишіть:

   ```bash
   dd if=/dev/zero of=/tmp/io_stress bs=10M count=500 oflag=dsync \
   & export MYPID=$!
   ```

#### Щоб додати процес/сценарій до `cgroup` вводу/виводу

1. Додайте PID попереднього процесу dd до елемента керування exercise_group `cgroup`. Впишіть:

   ```bash
   echo $MYPID | sudo tee /sys/fs/cgroup/exercise_group/cgroup.procs
   ```

#### Щоб підтвердити контроль використання ресурсів введення-виведення процесу

1. Перевірте використання вводу-виводу процесом, виконавши:

   ```bash
   iotop -p $MYPID
   ```

Вихідні дані відображатимуть швидкість читання/запису вводу/виводу для процесу io_stress.sh, яка не повинна перевищувати 1 МБ/с відповідно до обмеження.

#### Щоб видалити `cgroups`

1. Введіть наступні команди, щоб завершити будь-який фоновий процес, видалити непотрібну `cgroup` і видалити файл /tmp/io_stress.

   ```bash
   kill %1
   sudo rmdir /sys/fs/cgroup/exercise_group/
   sudo rm -rf /tmp/io_stress
   ```

## Завдання 8

### `taskset`

CPU affinity прив’язує конкретні процеси або потоки до певних ядер ЦП у багатоядерній системі. Ця вправа демонструє використання `taskset` для встановлення або отримання спорідненості ЦП процесу в Linux.

### Щоб дослідити CPU Affinity з `taskset`

1. Використовуйте `lscpu`, щоб отримати список доступних ЦП у вашій системі. Впишіть:

   ```bash
   lscpu | grep "On-line"
   ```

2. Давайте створимо зразок процесу за допомогою утиліти dd і збережемо його PID у змінній MYPID. Впишіть:

   ```bash
   dd if=/dev/zero of=/dev/null & export MYPID="$!"
   echo $MYPID
   ```

3. Отримайте поточну спорідненість для процесу `dd`. Впишіть:

   ```bash
   taskset -p $MYPID
   ```

   Вихід

   ```bash
   pid 1211483's current affinity mask: f
   ```

   Вихідні дані показують маску спорідненості ЦП процесу з PID 1211483 ($MYPID), представленим у шістнадцятковому форматі. У нашому прикладі системи відображається маска спорідненості «f», що зазвичай означає, що процес може виконуватися на будь-якому ядрі ЦП.

   !!! note "Примітка"

   ```
    Маска спорідненості ЦП "f" представляє конфігурацію, у якій увімкнено всі ядра ЦП. У шістнадцятковій системі числення «f» відповідає двійковому значенню «1111». Кожен біт у двійковому представленні відповідає ядру ЦП, причому «1» означає, що ядро ввімкнено та доступне для виконання процесу.

    Тому на чотириядерному процесорі з маскою "f":

    Core 0: Enabled
    Core 1: Enabled
    Core 2: Enabled
    Core 3: Enabled
   ```

### Щоб встановити/змінити схожість ЦП

1. Встановіть приналежність ЦП процесу dd до одного ЦП (ЦП 0). Впишіть:

   ```bash
   taskset -p 0x1 $MYPID
   ```

   Вихід

   ```bash
   pid 1211483's current affinity mask: f
   pid 1211483's new affinity mask: 1
   ```

2. Перевірте зміни, виконавши:

   ```bash
   taskset -p $MYPID
   ```

   Вихідні дані вказують на маску спорідненості ЦП процесу з PID $MYPID. Маска спорідненості дорівнює «1» у десятковій системі, що перекладається як «1» у двійковій системі. Це означає, що процес наразі прив’язаний до ядра ЦП 0.

3. Тепер встановіть приналежність ЦП процесу dd до кількох ЦП (ЦП 0 і 1). Впишіть:

   ```bash
   taskset -p 0x3 $MYPID
   ```

4. Видайте правильну команду `tasksel`, щоб перевірити останню зміну.

   ```bash
   taskset -p $MYPID
   ```

   На нашому демонстраційному 4-ядерному сервері ЦП результат показує, що маска спорідненості ЦП процесу дорівнює «3» (у десятковому вигляді). Це означає «11» у двійковій системі.

   !!! tip "Підказка"

   ```
    Десятковий "3" дорівнює "11" (або 0011) у двійковій системі.
    Кожна двійкова цифра відповідає ядру ЦП: ядро 0, ядро 1, ядро 2, ядро 3 (справа наліво).
    Цифра «1» на четвертій і третій позиціях (справа) означає, що процес може працювати на ядрах 0 і 1.
    Тому «3» означає, що процес прив’язаний до ядер ЦП 0 і 1.
   ```

5. Запустіть утиліту `top` або `htop` в окремому терміналі та спостерігайте, якщо ви бачите щось цікаве, експериментуючи з різними конфігураціями `taskset` для процесу.

6. Все готово. Використовуйте його PID ($MYPID), щоб припинити процес dd.

## Завдання 9

### `systemd-run`

Команда `systemd-run` створює та запускає тимчасові службові блоки для виконання команд або процесів. Вона також може запускати програми в блоках тимчасової області дії, блоках обслуговування, що запускаються шляхом, сокетом або таймером.

У цій вправі показано, як використовувати `systemd-run` для створення тимчасових службових одиниць у `systemd`.

#### Щоб запустити команду як тимчасову службу

1. Запустіть просту команду sleep 300 як тимчасову службу `systemd` за допомогою `systemd-run`. Впишіть:

   ```bash
   systemd-run --unit=mytransient.service --description="Example Service" sleep 300
   ```

2. Перевірте статус тимчасової служби за допомогою `systemctl status`. Впишіть:

   ```bash
   systemctl status mytransient.service
   ```

#### Щоб установити обмеження ресурсу пам’яті для тимчасової служби

1. Використовуйте параметр `--property` разом із `systemd-run`, щоб обмежити максимальне використання пам’яті для тимчасового процесу до 200 МБ. Впишіть:

   ```bash
   systemd-run --unit=mylimited.service --property=MemoryMax=200M sleep 300
   ```

2. Подивіться у відповідній файловій системі `cgroup` процес перевірки налаштування. Впишіть:

   ```bash
   sudo cat /sys/fs/cgroup/system.slice/mytransient.service/memory.max
   ```

   !!! tip "Підказка"

   ```
    `systemd.resource-control` — це конфігурація або керуюча сутність (концепція) у рамках `systemd`, призначена для керування та розподілу системних ресурсів для процесів і служб. А `systemd.exec` — це компонент `systemd`, який відповідає за визначення середовища виконання, у якому виконуються команди. Щоб переглянути різні параметри (властивості), які ви можете налаштувати під час використання systemd-run, зверніться до сторінок посібника `systemd.resource-control` і `systemd.exec`. Тут ви знайдете документацію для таких властивостей, як MemoryMax, CPUAccounting, IOWeight тощо. 
   ```

#### Щоб установити обмеження на ресурс ЦП для тимчасової служби

1. Давайте створимо тимчасовий блок `systemd` під назвою "myrealtime.service". Запустіть myrealtime.service із спеціальною політикою планування циклічного (rr) і пріоритетом. Впишіть:

   ```bash
   systemd-run --unit=myrealtime.service \
   --property=CPUSchedulingPolicy=rr --property=CPUSchedulingPriority=50 sleep 300
   ```

2. Перегляньте статус myrealtime.service. Крім того, захопіть/збережіть основний [sleep] PID у змінній під назвою MYPID. Впишіть:

   ```bash
   MYPID=$(systemctl status myrealtime.service   |  awk '/Main PID/ {print $3}')
   ```

3. Поки служба все ще працює, перевірте її політику планування ЦП. Впишіть:

   ```bash
   chrt  -p $MYPID
   pid 2553792's current scheduling policy: SCHED_RR
   pid 2553792's current scheduling priority: 50
   ```

### Щоб створити блок перехідного часу

1. Створіть простий таймер, який виконує просту команду echo. Параметр `--on-active=2m` встановлює таймер на спрацьовування через 2 хвилини після того, як блок таймера стане активним. Впишіть:

   ```bash
   systemd-run --on-active=2m --unit=mytimer.timer \
   --description="Example Timer" echo "Timer triggered"
   ```

   Таймер почне відлік часу з моменту активації пристрою і через 2 хвилини запустить вказану дію.

2. Переглянути деталі/статус щойно створеного таймера. Впишіть:

   ```bash
   systemctl status mytimer.timer
   ```

#### Щоб зупинити та очистити тимчасові блоки `systemd`

1. Введіть наступні команди, щоб переконатися, що різні тимчасові служби/процеси, запущені для цієї вправи, належним чином зупинені та видалені з вашої системи. Впишіть:

   ```bash
   systemctl stop mytransient.service
   systemctl stop mylimited.service
   systemctl stop myrealtime.service
   systemctl stop mytimer.timer
   ```

## Вправа 10

### `schedtool`

Ця вправа демонструє використання `schedtool` для розуміння та керування плануванням процесів у Rocky Linux. Ми також створимо сценарій для моделювання процесу для цієї мети.

#### Щоб встановити `schedtool`

1. Встановіть програму `schedtool`, якщо вона не встановлена на вашому сервері. Впишіть:

   ```bash
   sudo dnf -y install schedtool
   ```

#### Щоб створити імітований сценарій процесу

1. Створіть сценарій, який генерує навантаження на процесор для цілей тестування. Впишіть:

   ```bash
   cat > ~/cpu_load_generator.sh << EOF
   #!/bin/bash
   while true; do
        openssl speed > /dev/null 2>&1
        openssl speed > /dev/null 2>&1

   done
   EOF
   chmod +x ~/cpu_load_generator.sh
   ```

2. Запустіть сценарій у фоновому режимі. Впишіть:

   ```bash
   ~/cpu_load_generator.sh & echo $!
   ```

3. Зберіть PID для основного процесу `openssl`, запущеного в сценарії cpu_load_generator.sh. Зберігайте PID у змінній з іменем MYPID. Впишіть:

   ```bash
   export  MYPID=$(pidof openssl) ; echo $MYPID
   ```

#### Щоб використовувати `schedtool` для перевірки поточної політики планування

1. Використовуйте команду `schedtool`, щоб відобразити інформацію про планування процесу з PID $MYPID. Впишіть:

   ```bash
   schedtool $MYPID
   ```

   Вихід

   ```bash
   PID 2565081: PRIO   0, POLICY N: SCHED_NORMAL  , NICE   0, AFFINITY 0xf
   ```

#### Щоб використовувати `schedtool` для зміни політики планування

1. Змініть політику планування та пріоритет процесу FIFO та 10 відповідно. Впишіть:

   ```bash
   sudo schedtool -F -p 10 $!
   ```

2. Перегляньте ефект від змін. Впишіть:

   ```bash
   schedtool $MYPID
   ```

3. Змініть політику планування та пріоритет процесу на циклічний або SCHED_RR (RR) і 50 відповідно. Впишіть:

   ```bash
     sudo schedtool -R -p 50 $MYPID
   ```

4. Перегляньте ефект від змін. Впишіть:

   ```bash
   schedtool $MYPID
   ```

5. Змініть політику планування процесу на Idle або SCHED_IDLEPRIO (D). Впишіть:

   ```bash
   sudo schedtool -D $MYPID
   ```

6. Перегляньте ефект від змін.

7. Нарешті, скиньте політику планування процесу назад до початкового стандартного SCHED_NORMAL (N або інше). Впишіть:

   ```bash
   sudo schedtool -N $MYPID
   ```

#### Щоб завершити та очистити процес `cpu_load_generator.sh`

1. Все готово. Завершіть сценарій і видаліть сценарій `cpu_load_generator.sh`.

   ```bash
   kill $MYPID
   rm ~/cpu_load_generator.sh
   ```