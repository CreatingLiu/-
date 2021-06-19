#include <FastLED.h>
#include <TuyaWifi.h>

//hardware define
#define OUT_PIN 34
#define LED_NUMS 50
#define MIC_PIN A10

//dpid define
#define DPID_SWITCH 20
#define DPID_MODE   21
#define DPID_LIGHT  22
#define DPID_COLOR  24

//serial define
#define TUYA_SERIAL Serial1
#define LOG_SERIAL  Serial2

//datapoint value define
#define STATE_ON    0x01
#define STATE_OFF   0x00
#define MODE_WHITE  0x00
#define MODE_COLOUR 0x01
#define MODE_MIC   0x02

unsigned char pid[] = {"g2kypenftkitli6z"};
unsigned char mcu_ver[] = {"1.0.0"};
unsigned int led_light = 8;
#define COLOR_NUMS 7
CRGB colors[] = {
  CRGB::Red,
  CRGB::Orange,
  CRGB::Yellow,
  CRGB::Green,
  CRGB::LightGreen,
  CRGB::Blue,
  CRGB::Purple
};

//dp define
unsigned char dp_array[][2] =
{
  {DPID_SWITCH, DP_TYPE_BOOL},
  {DPID_MODE,   DP_TYPE_ENUM},
  {DPID_LIGHT,  DP_TYPE_VALUE},
  {DPID_COLOR,  DP_TYPE_STRING},
};

TuyaWifi my_device(&TUYA_SERIAL);
CRGB leds[LED_NUMS];
unsigned char state = STATE_OFF;
unsigned char mode = MODE_WHITE;

unsigned char led_state = LOW;
unsigned long last_time = 0;
unsigned char led_color = 0;

void setLED(CRGB data) {
  for(int i = 0; i < LED_NUMS; i++){
    leds[i] = data;
  }
}

void setup() {


  Serial.begin(115200);
  TUYA_SERIAL.begin(115200);
  LOG_SERIAL.begin(115200);
  my_device.init(pid, mcu_ver);
  my_device.set_dp_cmd_total(dp_array, sizeof(dp_array) / 2);
  my_device.dp_process_func_register(dp_process);
  my_device.dp_update_all_func_register(dp_update_all);

  FastLED.addLeds<WS2812, OUT_PIN, GRB>(leds, LED_NUMS);
  FastLED.setBrightness(led_light);

  last_time = millis();
  unsigned char value[1] = {STATE_ON};
  dp_process(DPID_SWITCH, value, 1);
}

void loop() {
  my_device.uart_service();

  while(LOG_SERIAL.available()){
    Serial.write(LOG_SERIAL.read());
  }

  if(state){
    switch(mode){
      case MODE_COLOUR:
        if(millis() - last_time > 50){
          last_time = millis();
          for(int i = LED_NUMS - 1; i > 0; i--){
            leds[i] = leds[i - 1];
          }
          leds[0] = colors[led_color];
          led_color = (led_color + 1) % COLOR_NUMS;
          FastLED.show();
        }
      break;
      case MODE_MIC:
        unsigned int lightNum = (exp(analogRead(MIC_PIN) / 675.84) - 1) * 1.5 * LED_NUMS / 1.7;
        for(int i = 0; i < lightNum && i < LED_NUMS; i++){
            leds[i] = CRGB(min(255 * i / (0.8 * LED_NUMS), 255), max(255 * (1 - i / (0.8 * LED_NUMS)), 0), 0);
        }
        for(int i = lightNum; i < LED_NUMS; i++){
            leds[i] = CRGB(0, 0, 0);
        }
        FastLED.show();
      break;
    }
  }
}

/**
 * @description: DP download callback function.
 * @param {unsigned char} dpid
 * @param {const unsigned char} value
 * @param {unsigned short} length
 * @return {unsigned char}
 */
unsigned char dp_process(unsigned char dpid,const unsigned char value[], unsigned short length) {
  switch(dpid){
    case DPID_SWITCH:
      state = value[0];
      my_device.mcu_dp_update(DPID_SWITCH, state, 1);
      if(state == STATE_OFF){
        setLED(CRGB(0, 0, 0));
        FastLED.show();
      }
      else{
        if(mode == MODE_WHITE){
          setLED(CRGB(255, 255, 255));
          FastLED.show();
        }
      }
    break;
    case DPID_MODE:
      mode = value[0];
      my_device.mcu_dp_update(DPID_MODE, mode, 1);
      if(state){
        if(mode == MODE_WHITE){
          setLED(CRGB(255, 255, 255));
          FastLED.show();
        }
      }
    break;
    case DPID_LIGHT:
      led_light = value[3];
      my_device.mcu_dp_update(DPID_LIGHT, led_light, 4);
      FastLED.setBrightness(led_light);
      FastLED.show();
    break;
  }
}

/**
 * @description: Upload all DP status of the current device.
 * @param {*}
 * @return {*}
 */
void dp_update_all(void) {
  my_device.mcu_dp_update(DPID_SWITCH, state, 1);
  my_device.mcu_dp_update(DPID_MODE, mode, 1);
  my_device.mcu_dp_update(DPID_LIGHT, led_light, 4);
}
