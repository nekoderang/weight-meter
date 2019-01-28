#include <OneWire.h>            //драйвер для датчиков темп DS
#include "HX711.h"              //драйвер для преобразователя тензодатчиков
//#include <SimpleTimer.h>
#include <Wire.h>               //для преобразователя i2c на LCD1602
#include <LiquidCrystal_I2C.h>  //библиотека для LCD1602

OneWire  ds(4);                 //Пин шины OneWire для темп. датчика

HX711 scale(A1, A0);            // Пины для преобразователя тензодатчиков

LiquidCrystal_I2C lcd(0x27,16,2);  // Устанавливаем дисплей

float calibration_factor = -0.81;          // калибровка для тензодатчиков
float units;                               // АЦП данные датчика
float ounces;                              // Данные в КГ
int weight;                                // Отмасштабированные данные для графика (0 - 16)
long h711_1;
int mode = 0;                              // Режим для красной кнопки (0 - вес и темпер; 1 - вес и график
int moderef = 0;                           // Режим для желтой кнопки (время обновления графика в режиме 1)
int buttonpin = 3;                         // Красная кнопка
int buttonpin1 = 2;                        // Желтая кнопка
int count_button = 0;                      // Счетчик для красной кнопки
int count_button1 = 0;                     // Счетчик для желтой кнопки
int graph0;                                // Шкала графика для 0-й строки
int graph1;                                // Шкала графика для 1-й строки
int x;                                     // Счетчик позиции каретки
int i;                                     // Для смещения значений массива (цикл for)
int b;
int colx[12] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11};  // Массив колонки 0 - 11
int agraph0[12];                                        // Массив шкалы 0-й строки с памятью в 12 значений
int agraph1[12];                                        // Массив шкалы 1-й строки с памятью в 12 значений
unsigned long tmr;                                      // Таймер
unsigned long reftmr;                                   // Таймер обновления графика
unsigned long tmr1;
                                                        
byte ic;                                                // темпер
byte datat[12];
byte addrt[8];
float celsius;


//Символы для графиков

byte char1[8] = {
  B00000, B00000, B00000, B00000, B00000, B00000, B00000, B11111,
};

byte char2[8] = {
  B00000, B00000, B00000, B00000, B00000, B00000, B11111, B11111,
};
 
byte char3[8] = {
  B00000, B00000, B00000, B00000, B00000, B11111, B11111, B11111,
};
 
byte char4[8] = {
  B00000, B00000, B00000, B00000, B11111, B11111, B11111, B11111,
};
 
byte char5[8] = {
  B00000, B00000, B00000, B11111, B11111, B11111, B11111, B11111,
};

byte char6[8] = {
  B00000, B00000, B11111, B11111, B11111, B11111, B11111, B11111,
};

byte char7[8] = {
  B00000, B11111, B11111, B11111, B11111, B11111, B11111, B11111,
};

byte char8[8] = {
  B11111, B11111, B11111, B11111, B11111, B11111, B11111, B11111,
};



void setup(void) 
{
  Serial.begin(9600);
                                             //Тензодатчики
  scale.set_scale();
  scale.tare();                              //Сбрасываем на 0 для установки тары
  scale.set_scale(calibration_factor);       //Применяем калибровку
                                             //Тип сигналов кнопок
  pinMode (buttonpin, INPUT);
  pinMode (buttonpin1, INPUT);
                                             //Параметры LCD экрана
  lcd.init();
  lcd.begin(16,2);                     
  lcd.backlight();
  lcd.createChar(1, char1);
  lcd.createChar(2, char2);
  lcd.createChar(3, char3);
  lcd.createChar(4, char4);
  lcd.createChar(5, char5);
  lcd.createChar(6, char6);
  lcd.createChar(7, char7);
  lcd.createChar(8, char8);

  lcd.setCursor(0, 0);
  lcd.print("Weight meter v.1");
  
  lcd.setCursor(0, 1);
  lcd.print("diy");
  delay(5000);
  lcd.clear();
   
}

void loop()
{

//Переключение режимов кнопкой
if (count_button == 0 && digitalRead(buttonpin) == HIGH) count_button = 1;
if (count_button == 1 && digitalRead(buttonpin) == LOW) 
  {
    mode++;
    if (mode == 2) mode = 0;
    count_button = 0;
    lcd.clear();
  }


switch (mode)
  {
    case 0: // Режим 0
    if (millis()-tmr > 2000)
     {
      tmr = millis();

      for(int i = 0;i < 10; i ++) units =+ scale.get_units(), 10;   // усредняем показания считав 10 раз 
      units / 10;                                                   // делим на 10
      if (units < 0)
      {
        units = 0.00;
      }
      ounces = units * 0.035274;
      
// Работа с датчиком температуры
      ds.search(addrt);
 
      ds.reset();
      ds.select(addrt);
      ds.write(0x44, 1);                                          // команда на измерение температуры

    //delay(1000);

      ds.reset();
      ds.select(addrt); 
      ds.write(0xBE);                                             // команда на начало чтения измеренной температуры

                                    // считываем показания температуры из внутренней памяти датчика
      for ( ic = 0; ic < 9; ic++) 
        {
          datat[ic] = ds.read();
        }

      int16_t raw = (datat[1] << 8) | datat[0];
      byte cfg = (datat[4] & 0x60);   // датчик может быть настроен на разную точность, выясняем её 
      if (cfg == 0x00) raw = raw & ~7; // точность 9-разрядов, 93,75 мс
        else if (cfg == 0x20) raw = raw & ~3; // точность 10-разрядов, 187,5 мс
          else if (cfg == 0x40) raw = raw & ~1; // точность 11-разрядов, 375 мс

      celsius = (float)raw / 16.0;  // преобразование показаний датчика в градусы Цельсия 

//Вывод данных на экран
      lcd.setCursor(0, 0);
      lcd.print("Weig: ");

      lcd.setCursor(0, 1);
      lcd.print("Temp: ");

      lcd.setCursor(6, 0);
      lcd.print(ounces);
      //lcd.setCursor(14, 0);
      //lcd.print("kg");

      lcd.setCursor(6, 1);
      lcd.print(celsius);
      lcd.setCursor(14, 1);
      lcd.print("C");
     }
     
    break;
    
    case 1: // Режим 1

      if (count_button1 == 0 && digitalRead(buttonpin1) == HIGH) count_button1 = 1;
      if (count_button1 == 1 && digitalRead(buttonpin1) == LOW) 
        {
          moderef++;
          if (moderef == 4) moderef = 0;
          count_button1 = 0;
        }
    
      switch (moderef)
      {
        case 0:             //10 сек
        reftmr = 10000;
        break;

        case 1:             //60 сек
        reftmr = 60000;
        break;

        case 2:             //15 мин
        reftmr = 900000;
        break;

        case 3:             //60 мин
        reftmr = 3600000;
        break;
      }
      
                                    //масштабирование значения веса к графическому отображению
      weight = map (ounces, 0, 30000, 1, 16); // !!!!!!Динамически выставляемые пределы
      graph1 = weight;
                                    //определение и решение проблемы превышения графической шкалы в линии
      if (weight > 8) 
        {
          graph1 = 8;
          graph0 = weight-8;
        }
      else
        {
          graph0 = 0;
        }
                                    //запись считанного значения с датчика в массив
      agraph0[x] = graph0;
      agraph1[x] = graph1;
                                    //отображение веса в режиме 1
      //lcd.setCursor(11, 0);
      //lcd.print(ounces);
      lcd.setCursor(12, 1);
      if (moderef == 0) lcd.print("10 s");    //отображение периода обновления графика
      if (moderef == 1) lcd.print("1  m");
      if (moderef == 2) lcd.print("15 m");
      if (moderef == 3) lcd.print("60 m");

      if (millis()-tmr1 > reftmr)
        {
          tmr1 = millis();
          
          for(int i = 0;i < 10; i ++) units =+ scale.get_units(), 10;   // усредняем показания считав 10 раз 
          units / 10;                                                   // делим на 10
          if (units < 0)
            {
              units = 0.00;
            }
          ounces = units * 0.035274;
          
          lcd.setCursor(11, 0);
          lcd.print(ounces);
          
          if (x >= 11) 
            {
              for (i = 0; i < 11 ; i++) 
                {
                  agraph0[i] = agraph0[i+1];
                  agraph1[i] = agraph1[i+1];
                  lcd.setCursor(colx[i], 1);
                  lcd.write(agraph1[i]);
              
                  if (agraph0[i] > 0) 
                    {
                      lcd.setCursor(colx[i], 0);
                      lcd.write(agraph0[i]);
                    }
                  else
                    {
                      lcd.setCursor(colx[i], 0);
                      lcd.print(" ");
                    }
                }
            }
          else 
            {
              lcd.setCursor(colx[x], 1);
              lcd.write(agraph1[x]);

              if (agraph0[x] > 0) 
                {
                  lcd.setCursor(colx[x], 0);
                  lcd.write(agraph0[x]);
                }
              else
                {
                  lcd.setCursor(colx[x], 0);
                  lcd.print(" ");
                }
              x++;
            }
        }
    break;
    
  }

 Serial.print("Weight = ");
 Serial.print(ounces, 1);
}
