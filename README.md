# Codigo-Robo
#include <Servo.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// LCD no endereço 0x27 com 16 colunas e 2 linhas
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Pinos dos botões
const int botaoAumenta = 2;
const int botaoDiminui = 3;
const int botaoTrocaa = 0;
const int botaoTrocab = 1;
const int botaoEnter = 9;
const int botaoPlay = 10;
const int deadMansTrigger = 4;
const int botaoEmergencia = 11;

// Pinos dos servos
const int pinoServo[] = { 5, 6, 7, 8 };
const int numServos = 4;
const int incremento = 5;

Servo servos[numServos];
int angulos[numServos] = { 90, 90, 90, 90 };
int servoAtivo = 0;

// Armazenar 5 posições (home, approachA, pick, approachB, place)
const int numPosicoes = 5;
int posicoes[numPosicoes][numServos];
int indicePosicaoSalvar = 0;

// === Protótipos de funções ===
void atualizarLCD();
void moverServoSuave(int index, int destino);

void setup() {
  pinMode(botaoAumenta, INPUT_PULLUP);
  pinMode(botaoDiminui, INPUT_PULLUP);
  pinMode(botaoTrocaa, INPUT_PULLUP);
  pinMode(botaoTrocab, INPUT_PULLUP);
  pinMode(botaoEnter, INPUT_PULLUP);
  pinMode(botaoPlay, INPUT_PULLUP);
  pinMode(deadMansTrigger, INPUT_PULLUP);
  pinMode(botaoEmergencia, INPUT_PULLUP);

  for (int i = 0; i < numServos; i++) {
    servos[i].attach(pinoServo[i]);
    servos[i].write(angulos[i]);
  }

  lcd.init();
  lcd.backlight();
  atualizarLCD();

  for (int i = 0; i < numPosicoes; i++) {
    for (int j = 0; j < numServos; j++) {
      posicoes[i][j] = 90;
    }
  }
}

void loop() {
  bool alterado = false;

  if (digitalRead(deadMansTrigger) == LOW) {
    if (digitalRead(botaoAumenta) == LOW) {
      if (angulos[servoAtivo] < 180) {
        angulos[servoAtivo] += incremento;
        servos[servoAtivo].write(angulos[servoAtivo]);
        alterado = true;
        delay(4);
      }
    }

    if (digitalRead(botaoDiminui) == LOW) {
      if (angulos[servoAtivo] > 0) {
        angulos[servoAtivo] -= incremento;
        servos[servoAtivo].write(angulos[servoAtivo]);
        alterado = true;
        delay(4);
      }
    }

    if (digitalRead(botaoTrocaa) == LOW) {
      servoAtivo--;
      if (servoAtivo < 0) servoAtivo = numServos - 1;
      alterado = true;
      delay(4);
    }

    if (digitalRead(botaoTrocab) == LOW) {
      servoAtivo++;
      if (servoAtivo >= numServos) servoAtivo = 0;
      alterado = true;
      delay(4);
    }
  }

  if (digitalRead(botaoEnter) == LOW) {
    for (int i = 0; i < numServos; i++) {
      posicoes[indicePosicaoSalvar][i] = angulos[i];
    }
    indicePosicaoSalvar++;
    if (indicePosicaoSalvar >= numPosicoes) {
      indicePosicaoSalvar = 0;
    }
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Posicao salva:");
    lcd.setCursor(0, 1);
    lcd.print(indicePosicaoSalvar == 0 ? 5 : indicePosicaoSalvar);
    delay(1000);
    atualizarLCD();
  }


if (digitalRead(botaoPlay) ==
