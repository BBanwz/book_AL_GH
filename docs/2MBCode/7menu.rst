Написание интерфейса на дисплее и управление с кнопок
-----------------------------------------------------

Подключение
~~~~~~~~~~~

Подключение дисплея продемонстрировано в предыдущем пункте. 

Кнопки соединяются последовательно по протоколу DXL. У каждой имеется свой айди, который можно определить через библиотеку ``DynamixelDevice`` со встроенным примером ``Console``.

Программирование интерфейса
~~~~~~~~~~~~~~~~~~~~~~~~~~~

  :: 
    
    #include <JsAr.h>   // Подключение библиотеки для работы с платой ESP.
    #include <DxlMaster2.h>       // Подключение библиотеки для работы с DXL-устройствами.
    #include <Wire.h>                     // Подключение библиотеки для работы с I2C устройствами.
    #include <LiquidCrystal_I2C.h>        // Подключение библиотеки для работы с LCD-дисплеем.

    #define NUM_BTNS 5

    uint8_t ids[NUM_BTNS] = {0x01,0x02,0x03,0x04,0x05};
    DynamixelDevice* btnDxl = (DynamixelDevice*)malloc(sizeof(DynamixelDevice) * NUM_BTNS);

    LiquidCrystal_I2C lcd(0x27, 16, 4);

    int init_buttons()
    {
      for(int i=0;i<NUM_BTNS;i++)
      {
        btnDxl[i] = DynamixelDevice(ids[i]);
        btnDxl[i].init();
        if (btnDxl[i].ping() != DYN_STATUS_OK)
          return ids[i];
      }
      return -1;
      
    }

    int read_buttons()
    {
      uint8_t btn;                                       // Переменные, необходимые для работы с кнопкой.

      for(int i=0;i<NUM_BTNS;i++)
      {
        btnDxl[i].read((uint8_t)27, (uint8_t)1, &btn); // Считывание регистра "нажатия" с кнопки.
        if (btn == 1)
        {
          delay(10);
          btnDxl[i].read((uint8_t)27, (uint8_t)1, &btn); // Считывание регистра "нажатия" с кнопки.
          if (btn ==1)
            return i;
        }
      }
      return -1;
    }

    int init_lcd()
    {
      byte count = 0;
      
      Wire.begin();
      for (byte i = 1; i < 120; i++)
      {
        Wire.beginTransmission (i);
        if (Wire.endTransmission () == 0)
          {
            if(i == 0x27)
            {
              count++;
              Serial.println("DISPLAY FOUND!");
              break;
            }
          delay (1);  
          } 
      } 

      if (count == 0)
        return 1;
        
      lcd.init();                                           // Инициализируем дисплей.
      lcd.backlight();                                      // Включаем подсветку
      lcd.setCursor(4.5, 1);                                // Устанавливаем курсор в середину 2 строки
      lcd.print("AGROLAB");                                 // Выводим текст
      delay(500);
      lcd.clear();  
      return -1;
    }


    typedef void(*Action)(); 

    class menu{
      private:
        String * menu_items;
        int selected_item;
        uint8_t num_items;
        Action   *actions;
        bool active; 
      public:
        menu(uint8_t n, String * items)
        {
          num_items = n - 1;
          menu_items = new String[n];
          actions = new Action[n];
          selected_item = 0;
          for(int i = 0; i<n;i++)
          {
            menu_items[i] = items[i];
            actions[i] = NULL;
          }
          active = false;
        }
        
        void bind_action(uint8_t n, Action act)
        {
          actions[n] = act;
        }
        
        void menu_down()
        {
          selected_item++;
          if (selected_item> num_items)
            selected_item = 0;
        }
        
        void menu_up()
        {
          selected_item--;
          if (selected_item < 0 )
            selected_item = num_items;
        }
        
        void menu_push()
        {
          if(actions[selected_item] == NULL)
            Serial.println("ACTION IS NOT BINDED TO THIS MENU ITEM");
          else
            actions[selected_item]();  
        }
        
        void draw_menu()
        { 
          if (active)
          { 
            lcd.clear();
            lcd.setCursor(0, selected_item % 4);
            lcd.print(char(126));                          
            int page_end = (selected_item / 4)*4 + 4 > num_items ? num_items % 4+1: 4;
            for(int i = 0; i<page_end; i++)  
            {                       
            lcd.setCursor(1, i);
            lcd.print(menu_items[(selected_item / 4)*4+i]);
            }
          }        
        }

        void set_active()
        {
          active = true;
        }
        
        void unset_active()
        {
          active = false;
        }
        
        bool get_active()
        {
          return active;
        }
    };

    #define MENU_MAIN_ITEMS 3
    String main_items[MENU_MAIN_ITEMS] = { "Controls", "Settings", "Calibration" };
    menu menu_main(MENU_MAIN_ITEMS, main_items);

    #define MENU_SUB_CONTROLS 5
    String controls_items[MENU_SUB_CONTROLS] = { "Web Set", "Airing Set", "LED Set", "Watering", "Back" };
    menu menu_controls(MENU_SUB_CONTROLS, controls_items);

    #define MENU_SUB_AIRING 4
    String airing_items[MENU_SUB_AIRING] = { "Time", "Humidity", "Button", "Back" };
    menu menu_airing(MENU_SUB_AIRING, airing_items);

    void enter_controls() {
      menu_main.unset_active();
      menu_controls.set_active();
    }
    void enter_settings() {
      Serial.println("Settings unavailable");
    }
    void enter_calibration() {
      Serial.println("Calibration unavailable");
    }
    void enter_airing() {
      menu_controls.unset_active();
      menu_airing.set_active();
    }

    void enter_web() {
      Serial.println("Web settings unavailable");
    }
    void enter_led() {
      Serial.println("LED settings unavailable");
    }
    void enter_watering() {
      Serial.println("Watering settings unavailable");
    }
    void back_main() {
      menu_main.set_active();
      menu_controls.unset_active();
    }

    void airing_time() {
      Serial.println("Airing is set to on time");
    }
    void airing_hum() {
      Serial.println("Airing is set to on humidity");
    }
    void airing_but() {
      Serial.println("Airing is set to on button");
    }
    void airing_back() {
      menu_controls.set_active();
      menu_airing.unset_active();
    }


    void setup() {
      JsAr.begin();            // Начинаем работу с платой ESP. Без этой строчки ничего работать не будет!
      DxlMaster.begin(57600);  // Начинаем работу с DXL-устройствами.
      Serial.begin(115200);

      Serial.println(String("ONBOARD VOLTAGE:") + JsAr.readVoltage());

      int status = init_buttons();
      if (status != -1) {
        Serial.println("BTN WITH ID " + String(ids[status]) + " NOT INITIALISED! Aborting.");
        ESP.restart();
      }

      status = init_lcd();
      if (status != -1) {
        Serial.println("LCD NOT INITIALISED! Aborting.");
        ESP.restart();
      }

      menu_main.bind_action(0, enter_controls);
      menu_main.bind_action(1, enter_settings);
      menu_main.bind_action(2, enter_calibration);
      menu_main.set_active();

      menu_controls.bind_action(0, enter_web);
      menu_controls.bind_action(1, enter_airing);
      menu_controls.bind_action(2, enter_led);
      menu_controls.bind_action(3, enter_watering);
      menu_controls.bind_action(4, back_main);

      menu_airing.bind_action(0, airing_time);
      menu_airing.bind_action(1, airing_hum);
      menu_airing.bind_action(2, airing_but);
      menu_airing.bind_action(3, airing_back);
    }

    void loop() {
      delay(100);
        Serial.println(menu_main.get_active());
          Serial.println(menu_controls.get_active());
            Serial.println(menu_airing.get_active());
            Serial.println("---------------------------");
      menu_main.draw_menu();
      menu_controls.draw_menu();
      menu_airing.draw_menu();
      
      switch (read_buttons()) {
        case 0:
          if(menu_main.get_active())
          {
            menu_main.menu_down();
          }else
          if(menu_controls.get_active())
          {
            menu_controls.menu_down();
          }else
          if(menu_airing.get_active())
          {
            menu_airing.menu_down();
          }
          break;
        case 1:
          if(menu_main.get_active())
          {
            menu_main.menu_up();
          }else
          if(menu_controls.get_active())
          {
            menu_controls.menu_up();
          }else
          if(menu_airing.get_active())
          {
            menu_airing.menu_up();
          }
          break;
        case 2:
          if(menu_main.get_active())
          {
            menu_main.menu_push();
          }
          else
          if(menu_controls.get_active())
          {
            menu_controls.menu_push();
          }else
          if(menu_airing.get_active())
          {
            menu_airing.menu_push();
          }
          break;
      }
    } 
