# Тема 1.2. Работа с файловыми системами. Часть 3

## Объем
2 часа

## Содержание лекции
- Основы работы с LVM.

## Введение в LVM (15 мин)
### Что такое LVM
LVM (Logical Volume Manager) - это система управления логическими томами, которая позволяет гибко управлять дисковым пространством, создавая виртуальные разделы, которые можно легко изменять в размере и перемещать.

### Преимущества LVM перед традиционными разделами
- **Гибкость**: Легко изменять размеры томов
- **Масштабируемость**: Добавление новых дисков
- **Снэпшоты**: Создание точек восстановления
- **Миграция**: Перемещение данных между дисками

### Основные компоненты LVM
- **Физические тома (PV)**: Физические диски или разделы
- **Группы томов (VG)**: Объединение физических томов
- **Логические тома (LV)**: Виртуальные разделы для использования

## Основные команды LVM (20 мин)
### Установка и настройка
```bash
# Установка LVM
sudo apt update
sudo apt install lvm2

# Проверка доступных дисков
lsblk
```

### Управление физическими томами
```bash
# Инициализация физического тома
sudo pvcreate /dev/sdb1

# Просмотр информации о PV
sudo pvs
sudo pvdisplay

# Удаление физического тома
sudo pvremove /dev/sdb1
```

### Управление группами томов
```bash
# Создание группы томов
sudo vgcreate vg_data /dev/sdb1 /dev/sdc1

# Просмотр информации о VG
sudo vgs
sudo vgdisplay

# Расширение группы томов
sudo vgextend vg_data /dev/sdd1

# Удаление группы томов
sudo vgremove vg_data
```

### Управление логическими томами
```bash
# Создание логического тома
sudo lvcreate -L 10G -n lv_home vg_data

# Просмотр информации о LV
sudo lvs
sudo lvdisplay

# Изменение размера логического тома
sudo lvresize -L +5G /dev/vg_data/lv_home

# Удаление логического тома
sudo lvremove /dev/vg_data/lv_home
```

## Создание LVM-структур (25 мин)
### Создание физических томов
```bash
# Инициализация нескольких дисков
sudo pvcreate /dev/sdb1 /dev/sdc1 /dev/sdd1

# Проверка результатов
sudo pvs
```

### Создание групп томов
```bash
# Создание группы с максимальным размером
sudo vgcreate vg_system /dev/sdb1 /dev/sdc1

# Создание группы с определенным размером
sudo vgcreate -s 32M vg_data /dev/sdd1
```

### Создание логических томов
```bash
# Создание тома с файловой системой
sudo lvcreate -L 20G -n lv_root vg_system
sudo mkfs.ext4 /dev/vg_system/lv_root

# Создание тома с определенным именем
sudo lvcreate -L 10G -n lv_var vg_data

# Создание тома с 100% свободного пространства
sudo lvcreate -l 100%FREE -n lv_backup vg_data
```

## Управление LVM (20 мин)
### Расширение/сжатие томов
```bash
# Расширение логического тома
sudo lvextend -L +5G /dev/vg_data/lv_home
sudo resize2fs /dev/vg_data/lv_home

# Сжатие логического тома
sudo lvreduce -L -5G /dev/vg_data/lv_home
sudo resize2fs /dev/vg_data/lv_home
```

### Снятие снэпшотов
```bash
# Создание снэпшота
sudo lvcreate -L 2G -s -n lv_home_snapshot /dev/vg_data/lv_home

# Восстановление из снэпшота
sudo umount /mnt/home
sudo lvconvert --merge /dev/vg_data/lv_home_snapshot

# Удаление снэпшота
sudo lvremove /dev/vg_data/lv_home_snapshot
```

### Миграция данных
```bash
# Перемещение данных между PV
sudo pvmove /dev/sdb1 /dev/sdc1

# Изменение размера PV
sudo pvresize /dev/sdb1
```

## Практическая часть (20 мин)
### Лабораторное задание
1. Создайте LVM-структуру:
   ```bash
   sudo pvcreate /dev/sdb1 /dev/sdc1
   sudo vgcreate vg_raid /dev/sdb1 /dev/sdc1
   sudo lvcreate -L 15G -n lv_data vg_raid
   ```
2. Создайте файловую систему и смонтируйте:
   ```bash
   sudo mkfs.ext4 /dev/vg_raid/lv_data
   sudo mkdir /mnt/lvm
   sudo mount /dev/vg_raid/lv_data /mnt/lvm
   ```
3. Создайте снэпшот:
   ```bash
   sudo lvcreate -L 2G -s -n lv_data_snapshot /dev/vg_raid/lv_data
   ```
4. Протестируйте восстановление:
   ```bash
   # Создать тестовые данные
   sudo dd if=/dev/zero of=/mnt/lvm/testfile bs=1M count=100
   # Восстановить из снэпшота
   sudo umount /mnt/lvm
   sudo lvconvert --merge /dev/vg_raid/lv_data_snapshot
   sudo mount /dev/vg_raid/lv_data /mnt/lvm
   ```

### Типичные проблемы и их решения
- **Ошибка "Cannot create volume group"**:
  Проверьте, что диски не используются другими системами
- **Ошибка "Insufficient free space"**:
  Убедитесь, что достаточно свободного места в VG
- **Проблемы с снэпшотами**:
  Выделите достаточное пространство для снэпшотов

## Заключение (10 мин)
### Ключевые моменты
- Понимание принципов работы LVM
- Умение создавать и управлять LVM-структурами
- Навыки работы со снэпшотами и миграцией данных

### Рекомендуемая литература
- Официальная документация LVM: https://tldp.org/HOWTO/LVM-HOWTO/
- Руководство по LVM: https://www.redhat.com/sysadmin/lvm-guide

### Вопросы для обсуждения
1. Когда стоит использовать LVM вместо традиционных разделов?
2. Какие ограничения у LVM?
3. Какие альтернативы LVM существуют?

## Чек-лист для самопроверки
- [ ] Понимание компонентов LVM
- [ ] Умение создавать LVM-структуры
- [ ] Навыки работы со снэпшотами
- [ ] Знание типичных проблем и их решений

## Глоссарий терминов
- **Физический том (PV)**: Физический диск или раздел
- **Группа томов (VG)**: Объединение PV
- **Логический том (LV)**: Виртуальный раздел
- **Снэпшот**: Точка восстановления

## Контрольные вопросы
1. Какие компоненты включает LVM?
2. Как создать снэпшот логического тома?
3. Какие команды используются для управления PV?

## Практическое задание на дом
1. Создайте LVM-структуру с несколькими LV
2. Настройте автоматическое создание снэпшотов
3. Изучите процесс миграции данных между PV

## Диаграмма слоев LVM
### Архитектура LVM
```mermaid
graph TB
    A[Физические устройства] --> B[Физические тома]
    B --> C[Группы томов]
    C --> D[Логические тома]
    D --> E[Файловые системы]
    E --> F[Монтирование]
    
    subgraph "Уровень 1: Физические устройства"
        A1[Диск /dev/sdb]
        A2[Диск /dev/sdc]
        A3[Диск /dev/sdd]
    end
    
    subgraph "Уровень 2: Физические тома"
        B1[PV /dev/sdb1]
        B2[PV /dev/sdc1]
        B3[PV /dev/sdd1]
    end
    
    subgraph "Уровень 3: Группы томов"
        C1[VG system]
        C2[VG data]
    end
    
    subgraph "Уровень 4: Логические тома"
        D1[LV root]
        D2[LV home]
        D3[LV var]
        D4[LV backup]
    end
    
    subgraph "Уровень 5: Файловые системы"
        E1[ext4]
        E2[xfs]
        E3[btrfs]
    end
    
    subgraph "Уровень 6: Монтирование"
        F1[/]
        F2[/home]
        F3[/var]
        F4[/backup]
    end
```
