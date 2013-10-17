---
author: Ian
comments: true
date: 2010-12-30 21:48:20+00:00
layout: post
slug: arduino-8x8-game-of-life
title: Arduino Game Of Life on 8x8 LED Matrix
alias: blog/archives/170/index.html
wordpress_id: 170
categories:
- microprocessors
- programming
- projects
tags:
- arduino
- AVR
- led matrix
- Programming
---

I was messing around with some Christmas toys and threw together a [Conway's Game Of Life](http://en.wikipedia.org/wiki/Conway's_Game_of_Life) implementation together on my Arduino.  I just love how quick it is to get things up and running on this platform.  It took me longer to solder a platform for the LED matrix to raise it up off the breadboard so the wires would all fit than the whole rest of the project.

Here's a [video of Arduino Game Of Life](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0965.mp4) running.

Anyway, more pictures and full source code are below.  The code has a couple of conditionally compiled options, one for storing rand seeds to EEPROM.  With the randomization turned on, every so often I'd see a "game" that progressed really nicely.  I wanted to be able to go back and watch the same game again.

[![](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0956-150x150.jpg)](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0956.jpg) [![](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0964-150x150.jpg)](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0964.jpg)





## <!-- more -->



The LED matrix conveniently has a non-standard pin spacing (between the rows of pins that is), and is as wide as the full working area on a solderless breadboard.  I quickly knocked up a breakout board for it, mounting some header pins on the bottom, and the matrix on the top.  This way it plugs directly into the breadboard, leaving clearance for the wires below.

[![](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0950-150x150.jpg)](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0950.jpg) [![](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0951-150x150.jpg)](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0951.jpg) [![](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0952-150x150.jpg)](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0952.jpg) [![](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0955-150x150.jpg)](http://brownsofa.org/blog/wp-content/uploads/2010/12/IMG_0955.jpg)

**Source code:**

    
    #include <EEPROM.h>
    
    /*
    GAME OF LIFE
    For an 8x8 LED matrix
    
    Connect digital IO pins 2-13 and analog pins 0-3 to LED matrix (through current limiting resistors)
    Analog pin 4 (see RANDOMIZER_ANALOG_PIN) is used to select the randomization mode.
      - connect to 5V to randomize and generate a new rand seed
      - connect to 3.3V to use the last randomized set (USE_EEPROM must be #defined)
      - connect to 0V to use the hard coded starting state
    Analog pin 5 (see UNCONNECTED_ANALOG_PIN) is left unconnected and used to add a bit of true randomness to seed the RNG
     */
    
    const int NUMROWS = 8;
    const int NUMCOLS = 8;
    const int MICROS = 100;
    
    const int EEPROM_ADDRESS_1 = 34;
    const int EEPROM_ADDRESS_2 = 35;
    
    /**
     * The "rows" in my LED matrix (http://www.sparkfun.com/products/681) are the ground connections.
     * They need to either be pulled low to enable the row, or be set to high impedance (input) mode.
     */
    const byte rows[NUMROWS] = { 15, 7, 6, 5, 2, 3, 4, 14 };
    
    /**
     * I'm using only one of the colors in the LED matrix.
     */
    const byte cols[NUMCOLS] = { 17, 13, 12, 11, 8, 9, 10, 16 };
    
    const int RANDOMIZER_ANALOG_PIN = 4;
    const int UNCONNECTED_ANALOG_PIN = 5;
    
    /**
     * Conditional compilation directives
     */
    //#define USE_EEPROM 1                // uncomment this line to enable eeprom storage
    //#define SHOW_STARTUP_SEQUENCE 1     // uncomment this line to enable the startup sequence
    
    boolean gameBoard[NUMROWS][NUMCOLS] = {
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 1, 1, 0, 0, 0, 0 },
      { 0, 1, 1, 0, 0, 0, 0, 0 },
      { 0, 0, 1, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 }
    };
    
    boolean newGameBoard[NUMROWS][NUMCOLS] = {
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 },
      { 0, 0, 0, 0, 0, 0, 0, 0 },
    };
    
    /////////////////////////////////
    
    void setup() {
      Serial.begin(9600);
      Serial.println("\nBegin setup()");
    
    #ifdef USE_EEPROM
      Serial.println("EEPROM code enabled");
    #else
      Serial.println("EEPROM code disabled");
    #endif // defined USE_EEPROM
    
      allOff();
      startupSequence();
      setUpInitialBoard();
      Serial.println("End setup()\n");
    }
    
    void loop() {
      long time = millis();
    
      // Display the current game board for approx. 250ms
      do {
        displayGameBoard();
      }
      while (millis() < time + 250);
    
      // Calculate the next iteration
      calculateNewGameBoard();
      swapGameBoards();
    }
    
    ///////////////////////////////////
    
    /**
     * Turns off all LEDs, and initializes the pin modes correctly.
     */
    void allOff() {
      for (int i=0; i<NUMROWS; i++) {
        pinMode(rows[i], INPUT);
      }
      for (int i=0; i<NUMCOLS; i++) {
        pinMode(cols[i], OUTPUT);
        digitalWrite(cols[i], LOW);
      }
    }
    
    /**
     * Does some randomizing of the initial board state.
     */
    void setUpInitialBoard() {
      // Generate a new seed for the RNG
      int seed = analogRead(UNCONNECTED_ANALOG_PIN);
    
      // Look at how the randomizer pin is connected.
      // If it's pulled high, then generate and store a new seed.
      // If it's middle (3.3v) then read the seed from EEPROM (if that code is enabled with USE_EEPROM)
      pinMode(RANDOMIZER_ANALOG_PIN, INPUT);
      int randomizerPinValue = analogRead(RANDOMIZER_ANALOG_PIN);
      if (randomizerPinValue > 900) {  // connected to +5V
    #ifdef USE_EEPROM
        // Generate and store a new random seed
        Serial.println("Generating new randseed...");
        Serial.print("Storing ");
        Serial.print(seed, DEC);
        EEPROM.write(EEPROM_ADDRESS_1, lowByte(seed));
        EEPROM.write(EEPROM_ADDRESS_2, highByte(seed));
        Serial.print("... done\n");
    #endif // defined USE_EEPROM
    
        Serial.print("Seeding RNG with ");
        Serial.print(seed, DEC);
        Serial.print("\n");
    
        randomSeed(seed);
        perturbInitialGameBoard();
      } else if (randomizerPinValue > 300) {  // connected to +3.3V
        // Retrieve random seed from EEPROM
    #ifdef USE_EEPROM
        Serial.println("Retrieving randseed...");
        int hi = EEPROM.read(EEPROM_ADDRESS_2);
        int lo = EEPROM.read(EEPROM_ADDRESS_1);
        seed = (hi << 8 ) | lo;
        Serial.print("Read ");
        Serial.print(seed, DEC);
        Serial.print("\n");
    #endif // defined USE_EEPROM
    
        Serial.print("Seeding RNG with ");
        Serial.print(seed, DEC);
        Serial.print("\n");
    
        randomSeed(seed);
        perturbInitialGameBoard();
      } else {
        Serial.println("Using basic board.");
      }
    }
    
    /**
     * Does a nice little animation to aid visual checks that all LEDs are correctly connected and operating.
     * Lights every LED in each row, going back and forth across the rows, from top to bottom.
     */
    void startupSequence() {
    #ifdef SHOW_STARTUP_SEQUENCE
      for (int row=0; row<NUMROWS; row++) {
        if (row%2 == 0) {
          for (int col=0; col<NUMCOLS; col++) {
            setLedOn(row, col);
            delay(25);
            setLedOff(row, col);
          }
        } else {
          for (int col=NUMCOLS-1; col>=0; col--) {
            setLedOn(row, col);
            delay(25);
            setLedOff(row, col);
          }
        }
      }
    #endif // defined SHOW_STARTUP_SEQUENCE
    }
    
    /**
     * Makes a small number of random changes to the game board
     */
    void perturbInitialGameBoard() {
      int numChanges = random(20,100);
      for (int i=0; i<numChanges; i++) {
        int row = random(0, NUMROWS);
        int col = random(0, NUMCOLS);
        gameBoard[row][col] = !gameBoard[row][col]; // toggle the led in this position
      }
    }
    
    /**
     * Loops over all game board positions, and briefly turns on any LEDs for "on" positions.
     */
    void displayGameBoard() {
      for (byte row=0; row<NUMROWS; row++) {
        for (byte col=0; col<NUMCOLS; col++) {
          if (gameBoard[row][col]) {
            pulseLed(row, col);
          }
        }
      }
    }
    
    /**
     * Turns on the specified LED for a v. short period of time
     */
    void pulseLed(byte row, byte col) {
      setLedOn(row, col);
      delayMicroseconds(MICROS);
      setLedOff(row, col);
    }
    
    /**
     * Counts the number of active cells surrounding the specified cell.
     * Cells outside the board are considered "off"
     * Returns a number in the range of 0 <= n < 9
     */
    byte countNeighbors(byte row, byte col) {
      byte count = 0;
      for (char rowDelta=-1; rowDelta<=1; rowDelta++) {
        for (char colDelta=-1; colDelta<=1; colDelta++) {
          // skip the center cell
          if (!(colDelta == 0 && rowDelta == 0)) {
            if (isCellAlive(rowDelta+row, colDelta+col)) {
              count++;
            }
          }
        }
      }
      return count;
    }
    
    /**
     * Returns whether or not the specified cell is on.
     * If the cell specified is outside the game board, returns false.
     */
    boolean isCellAlive(char row, char col) {
      if (row < 0 || col < 0 || row >= NUMROWS || col >= NUMCOLS) {
        return false;
      }
    
      return (gameBoard[row][col] == 1);
    }
    
    /**
     * Encodes the core rules of Conway's Game Of Life, and generates the next iteration of the board.
     * Rules taken from wikipedia.
     */
    void calculateNewGameBoard() {
      for (byte row=0; row<NUMROWS; row++) {
        for (byte col=0; col<NUMCOLS; col++) {
          byte numNeighbors = countNeighbors(row, col);
    
          if (gameBoard[row][col] && numNeighbors < 2) {
            // Any live cell with fewer than two live neighbours dies, as if caused by under-population.
            newGameBoard[row][col] = false;
          } else if (gameBoard[row][col] && (numNeighbors == 2 || numNeighbors == 3)) {
            // Any live cell with two or three live neighbours lives on to the next generation.
            newGameBoard[row][col] = true;
          } else if (gameBoard[row][col] && numNeighbors > 3) {
            // Any live cell with more than three live neighbours dies, as if by overcrowding.
            newGameBoard[row][col] = false;
          } else if (!gameBoard[row][col] && numNeighbors == 3) {
            // Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.
            newGameBoard[row][col] = true;
          } else {
            // All other cells will remain off
            newGameBoard[row][col] = false;
          }
        }
      }
    }
    
    /**
     * Copies the data from the new game board into the current game board array
     */
    void swapGameBoards() {
      for (byte row=0; row<NUMROWS; row++) {
        for (byte col=0; col<NUMCOLS; col++) {
          gameBoard[row][col] = newGameBoard[row][col];
        }
      }
    }
    
    void setLed(byte row, byte col, boolean level) {
      if (level == HIGH) {
        pinMode(rows[row], OUTPUT);
        digitalWrite(cols[col], HIGH);
      }
      else if (level == LOW) {
        pinMode(rows[row], INPUT);
        digitalWrite(cols[col], LOW);
      }
    }
    
    void setLedOn(byte row, byte col) {
      setLed(row, col, HIGH);
    }
    void setLedOff(byte row, byte col) {
      setLed(row, col, LOW);
    }
