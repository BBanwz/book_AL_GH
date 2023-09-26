Датчик освещенности
===================

Датчики освещенности — это электронные устройства, которые обнаруживают и измеряют интенсивность света в непосредственной близости от них. Они имеют широкий спектр применения в различных отраслях, включая автомобилестроение, бытовую электронику, промышленную автоматизацию и системы умного дома. PolusLab, известная технологическая компания, разработала передовой датчик освещенности, который обеспечивает исключительную точность, надежность и эффективность.

Принцип работы:
~~~~~~~~~~~~~~~

Датчик света  использует чувствительный механизм на основе фотодиода. ``Фотодиоды`` — это полупроводниковые устройства, преобразующие световую энергию в электрический ток. При воздействии света фотодиод генерирует ток, пропорциональный интенсивности падающего света. Схема датчика освещенности усиливает этот ток и преобразует его в ``сигнал напряжения``, который можно дополнительно обрабатывать для различных приложений.


::

  #include <JsAr.h>
  #include <DxlMaster.h>
  #include <Wire.h>
  #include <iarduino_I2C_DSL.h>
  iarduino_I2C_DSL dsl;                             
                                                    
  void setup(){
      JsAr.begin();
      DxlMaster.begin(57600);
      delay(500);
      Serial.begin(9600);
      while(!Serial){;}
      dsl.begin();
  }

  void loop(){
      Serial.print("Освещённость = ");
      Serial.print( dsl.getLux() );
      Serial.print(" лк.\t");
      Serial.print("Мерцание = ");
      Serial.print( dsl.getPulsation() );
      Serial.print("%\t");
      Serial.print("Близость = ");
      Serial.print( dsl.getProximity() );
      Serial.print(".\r\n");
      delay(500);
  }

.. raw:: html

    <div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto;">
        <iframe src="https://www.youtube.com/embed/ZAADjJ0k2Jc?si=13wQXEiegYbrjV13" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
    </div>
