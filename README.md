


Annamalaiece/Annamalaiece is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
include "FastLED.h"

// Matrix size
#define NUM_ROWS 16
#define NUM_COLS 16
#define WIDTH NUM_COLS
#define HEIGHT NUM_ROWS
#define NUM_LEDS NUM_ROWS * NUM_COLS
#define MATRIX_TYPE 1
// LEDs pin
#define DATA_PIN 3
// LED brightness
#define BRIGHTNESS 255
// Define the array of leds
CRGB leds[NUM_LEDS];


//Drop
//@stepko
#define Sat (255)
#define Hue (150)
byte type;
#define WIDTH NUM_COLS
#define HEIGHT NUM_ROWS

void fillAll(CRGB color) {
  for (int i = 0; i < NUM_LEDS; i++) {
    leds[i] = color;
  }
}
void drawPixelXY(uint8_t x, uint8_t y, CRGB color)
{
  if (x < 0 || x > (WIDTH - 1) || y < 0 || y > (HEIGHT - 1)) return;
  uint32_t thisPixel = XY((uint8_t)x, (uint8_t)y);
  leds[thisPixel] = color;
}

bool loadingFlag = true;
CRGBPalette16 currentPalette(PartyColors_p);
uint8_t colorLoop = 1;
uint8_t ihue = 0;
uint8_t hue;
CRGB color;

void N(){
   for (byte y = 0; y < NUM_ROWS; y++) {
        for (byte x = 0; x < NUM_COLS; x++) {
          //uint16_t pixelHue = inoise16 ((uint32_t)x*5000, (uint32_t)y*5000+ms*10,ms*20);  
          uint8_t pixelHue8 = inoise8 (x*30,y*30,millis()/16);
 leds[XY(x, y)] = ColorFromPalette(currentPalette, pixelHue8);}
   }blur2d(leds,NUM_COLS, NUM_ROWS, 32 );}

 void PoolNoise()
{ 
  if (loadingFlag)
  {loadingFlag = false;
    hue = Hue;
  }
  fill_solid( currentPalette, 16, CHSV(hue,Sat,230));
    currentPalette[10] = CHSV(hue,Sat-60,255);
    currentPalette[9] = CHSV(hue,255-Sat,210);
    currentPalette[8] = CHSV(hue,255-Sat,210);
    currentPalette[7] = CHSV(hue,Sat-60,255);
    if(Hue == 1){
    hue++;}
  blur2d(leds, NUM_COLS, NUM_ROWS, 100);
  N();
}

void drawCircle(int x0, int y0, int radius, const CRGB &color) {
  int a = radius, b = 0;
  int radiusError = 1 - a;

  if (radius == 0) {
    drawPixelXY(x0, y0, color);
    return;
  }
  while (a >= b)  {
    drawPixelXY(a + x0, b + y0, color);
    drawPixelXY(b + x0, a + y0, color);
    drawPixelXY(-a + x0, b + y0, color);
    drawPixelXY(-b + x0, a + y0, color);
    drawPixelXY(-a + x0, -b + y0, color);
    drawPixelXY(-b + x0, -a + y0, color);
    drawPixelXY(a + x0, -b + y0, color);
    drawPixelXY(b + x0, -a + y0, color);
    b++;
    if (radiusError < 0)
      radiusError += 2 * b + 1;
    else
    {
      a--;
      radiusError += 2 * (b - a + 1);
    }
  }
}

int rad[(HEIGHT+WIDTH)/8];
byte posx[(HEIGHT+WIDTH)/8],posy[(HEIGHT+WIDTH)/8];

void drop() {

  if (loadingFlag)
  { loadingFlag = false;
    hue = Hue;
    for (int i = 0; i < ((HEIGHT + WIDTH) / 8) - 1; i++)
    {
      posx[i] = random(WIDTH);
      posy[i] = random(HEIGHT);
      rad[i] = random(-1, (HEIGHT + WIDTH) / 2);
    }
  }
  fill_solid( currentPalette, 16, CHSV(hue, Sat, 230));
  currentPalette[10] = CHSV(hue, Sat - 60, 255);
  currentPalette[9] = CHSV(hue, 255 - Sat, 210);
  currentPalette[8] = CHSV(hue, 255 - Sat, 210);
  currentPalette[7] = CHSV(hue, Sat - 60, 255);
  fillAll(ColorFromPalette(currentPalette, 1));
  for (uint8_t i = 0; i < ((HEIGHT + WIDTH) / 8) - 1; i++)
  {
    drawCircle(posx[i], posy[i], rad[i], ColorFromPalette(currentPalette, (256 / 16) * 8.5 - rad[i]));
    drawCircle(posx[i], posy[i], rad[i] - 1, ColorFromPalette(currentPalette, (256 / 16) * 7.5 - rad[i]));
    if (rad[i] >= (HEIGHT + WIDTH) / 2) {
      rad[i] = -1;
      posx[i] = random(WIDTH);
      posy[i] = random(HEIGHT);
    }
    else
      rad[i]++;

  }
  if (Hue == 0)
    hue++;
  blur2d(leds, NUM_COLS, NUM_ROWS, 32 );
}

void draw() {
  switch (type) {
    case 0: PoolNoise(); break;
    case 1: drop(); break;
  }
}

void setup() {
  //Serial.begin(250000);
  FastLED.addLeds<NEOPIXEL, DATA_PIN>(leds, NUM_LEDS);
  FastLED.setBrightness(BRIGHTNESS);
  pinMode(2, INPUT_PULLUP);
}

void loop() {
  bool buttonPressed = digitalRead(2) == LOW;
  draw();
  if (buttonPressed) {
    if (type >= 1) {
      type = 0;
    } else {
      type += 1;
    }
    FastLED.clear();
    loadingFlag = true;
    delay(100);
  } else {
    FastLED.show();
    FastLED.delay(1000 / 60);
  }
  //static int frame = 0;
  //if (frame++ % 32 == 0)
  // Serial.println(FastLED.getFPS());

} //loop


uint16_t XY (uint8_t x, uint8_t y) {
  return (y * NUM_COLS + x);
}
