Чтение данных с датчиков PolusLab
=================================

.. raw:: html

    <div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto;">
        <iframe src="https://www.youtube.com/embed/U_ttxNbKHyg?si=8T0grcM0K2Ejsa6R" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
    </div>

Этот код предназначен для Arduino и используется для управления и получения данных с датчиков PolusLab. Вот его функции, подробно описанные по пунктам:

1. Подключение библиотек: Включение библиотек DxlMaster2 (Dynamixel) для управления Dynamixel устройствами и JsAr для работы с Ардуино.

2. Определение констант и переменных:

- ``LFS_TEMPERATURE_ID``, ``LFS_CTRL_MEASURE_START_STOP_REG``, ``LFS_SENSOR_EN_REG`` и ``LFS_DATA_REG`` определены как адреса регистров для идентификации температуры, контроля измерений, включения датчика и данных датчика соответственно.

- ``LFS_ADDRESS`` - адрес устройства.

- dynamixel_baudrate и serial_baudrate определяют скорость передачи данных для Dynamixel и Serial соответственно.

- Используется объект DynamixelDevice с именем "device."

- Создается DynamixelStatus с именем "status."


3. Функция setup инициализирует настройки и связь:

- Инициализация JsAr с вызовом ``JsAr.begin()``.

- Инициализация Dynamixel с ``DxlMaster.begin()`` и установка таймаута.

- Инициализация Serial с вызовом ``Serial.begin()``.

- Установка протокола версии для устройства с вызовом ``device.protocolVersion(2)``.

- Проверка связи с устройством, отправка ping и вывод состояния на Serial.

- Остановка измерений, очистка списка датчиков, добавление датчика температуры в список, и запуск измерений с использованием команд ``write`` и задержек между ними.


4. Функция loop выполняется постоянно и считывает температуру:

- Записывает данные с датчика PolusLab в переменную data с помощью вызова ``device.read()``.

- Выводит данные о температуре на Serial, предварительно преобразовав значение в градусы Цельсия.

- Задает задержку в 3 секунды перед следующим считыванием данных.

В общем, этот код инициализирует и настраивает связь с датчиками через Dynamixel, считывает данные и выводит информацию на Serial::
   
    #include <DxlMaster2.h>
    #include <JsAr.h>

    #define LFS_TEMPERATURE_ID 220

    #define LFS_CTRL_MEASURE_START_STOP_REG 24  
    #define LFS_SENSOR_EN_REG 60
    #define LFS_DATA_REG 80

    #define LFS_ADDRESS 206

    DynamixelDevice device(LFS_ADDRESS);

    const unsigned long dynamixel_baudrate = 115200;
    const unsigned long serial_baudrate = 115200;

    DynamixelStatus status = 0; 


    void setup() {
        JsAr.begin();
        DxlMaster.begin(dynamixel_baudrate);
        DxlMaster.setTimeOut(10);
        Serial.begin(serial_baudrate);
        device.protocolVersion(2);

        status = device.ping(); 
        if (status == DYN_STATUS_OK)
            Serial.println("Ping status: Ok!");
        else
            Serial.println("Ping status: Err!");

        uint16_t sensors_en_buf[10] = {0};
        device.write(LFS_CTRL_MEASURE_START_STOP_REG, 0);
        delay(500);
        device.write(LFS_SENSOR_EN_REG, 20, (uint8_t*) sensors_en_buf);
        delay(500);
        sensors_en_buf[0] = LFS_TEMPERATURE_ID;
        device.write(LFS_SENSOR_EN_REG, 20, (uint8_t*) sensors_en_buf);
        delay(500);
        device.write(LFS_CTRL_MEASURE_START_STOP_REG, 1);
        delay(500);
    }

    void loop() {
        uint16_t data = 0;
        device.read(LFS_DATA_REG, data); 
        Serial.println("temperature = " + String(data/10));
        delay(3000);
    }


