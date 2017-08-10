# [DC25 HHV Reverse Engineering Challenge](https://github.com/DCHHV/DC25_HHV_RE)

## Instructions

Once you have a kit in-hand, (if you don't yet, go to the HHV and pick one up!) there are two steps to be aware of for this challenge.

The first challenge, is to reverse engineer the schematic of the device itself.  Grab a piece of paper, draw up the schematic, and take it to the HHV.  A correct schematic will earn you a small prize (while available).

The meat of the challenge is the function of the device.  It's locked, find the passcode to unlock it; when unlocked the green LED will remain solid.  There are multiple ways to accomplish this task.  If you are stuck or just not sure how to proceed, come on down to the HHV and ask for help.  Once the device has been unlocked, make your way back to the HHV again to collect a larger prize (while available).

This challenge is meant to be fun for all levels of experience, but is designed to be an introduction to reverse engineering and a demonstration in how insecure some devices really are.

## After the con

After DEF CON 25 is over, the full sources for this challenge will be released as well as a write up of the full challenge.

# Solution

## Schematics
### Original Schematics

### Schematic for interfacing with an Arduino


## Attacks
### Brute Force
The following is an Arduino Sketch to implement a brute force attack on the RE Kit.
	
	/* Author: Onkar Raut (@rautonkar) */
	/* HINT to improve performance: Toggle Count is variable between 100 to 108*/
	
	/* The Key is generated from this number */
	uint16_t number = 0;
	
	/* Measures the number of times a FALLING edge is observed on PIC12F1572 Pin 3 */
	uint8_t toggle_count = 0;
	
	/* A printable sequence of the Button Presses applied by the Arduino */
	uint8_t sequence[8];
	
	/* Arduino Pin connected to PIC12F1572 Pin 3 */
	const byte LED_PIC_UNLOCK = 2;

	const byte NPN_ON = HIGH;
	const byte NPN_OFF = LOW;
	const byte PNP_ON = LOW;
	const byte PNP_OFF = HIGH;

	/* The following are the Arduino pins connected to transistors operating as switches. */
	/* When holding PCB in a readable format*/
	/* BUTTON 1 is first on the Left */
	const byte BUTTON_1 = 4;
	const byte BUTTON_2 = 7;
	const byte BUTTON_3 = 8;
	const byte BUTTON_4 = 12;

	/**
	 * Create a key using a 16-bit integer.
	 * Key has 8 bins. Each bin can have a number 1,2,3, or, 4.
	 * Key is derived from the number input as the second argument.
	 */
	void make_key(uint8_t * key, uint16_t number)
	{
		key[0] = ((number >> 0)& 0x3)  + 1;
		key[1] = ((number >> 2)& 0x3)  + 1;
		key[2] = ((number >> 4)& 0x3)  + 1;
		key[3] = ((number >> 6)& 0x3)  + 1;
		key[4] = ((number >> 8)& 0x3)  + 1;
		key[5] = ((number >> 10)& 0x3) + 1;
		key[6] = ((number >> 12)& 0x3) + 1;
		key[7] = ((number >> 14)& 0x3) + 1;

		return;
	}
	/**
	 * 1. Write key value to port pin.
	 * 2. Delay for 100ms.
	 * 3. Release port pin.
	 * 4. Delay for 100ms.
	 * 5. Move to next key.
	 */
	void apply_key(uint8_t * key, size_t key_len)
	{
		for(size_t itr = 0; itr < key_len; itr++)
		{
			if (1 == key[itr])
			{
			  digitalWrite(BUTTON_1, NPN_ON);
			  delay(100);
			  digitalWrite(BUTTON_1, NPN_OFF);
			  delay(100);
			}
			else if (2 == key[itr])
			{
			  digitalWrite(BUTTON_2, NPN_ON);
			  delay(100);
			  digitalWrite(BUTTON_2, NPN_OFF);
			  delay(100);
			}
			else if (3 == key[itr])
			{
			  digitalWrite(BUTTON_3, PNP_ON);
			  delay(100);
			  digitalWrite(BUTTON_3, PNP_OFF);
			  delay(100);
			}
			else if (4 == key[itr])
			{
			  digitalWrite(BUTTON_4, PNP_ON);
			  delay(100);
			  digitalWrite(BUTTON_4, PNP_OFF);
			  delay(100);
			}
			else
			{
				;
			}
		}
	}

	void printCombination(uint8_t const * const key, uint16_t num){
		if(Serial)
		{      
		  Serial.println("Trying Combination:");
		  Serial.print(key[0]);
		  Serial.print(",");
		  Serial.print(key[1]);
		  Serial.print(",");
		  Serial.print(key[2]);
		  Serial.print(",");
		  Serial.print(key[3]);
		  Serial.print(",");
		  Serial.print(key[4]);
		  Serial.print(",");
		  Serial.print(key[5]);
		  Serial.print(",");
		  Serial.print(key[6]);
		  Serial.print(",");
		  Serial.print(key[7]);
		  Serial.print("; Number:");
		  Serial.print(num, HEX);
		  Serial.println(";");
		}
	}

	/**
	 * Function to execute when PIC12F1572 pin 3 toggles.
	 * Increments the toggle count.
	 */
	void pic_led_toggled(void)
	{
	  toggle_count++;
	}

	void setup() {
	  /* Setup Communications with a PC */
	  Serial.begin(9600);
	 
	  /* Set up the pins acting as button presses */
	  pinMode(BUTTON_1, OUTPUT);
	  pinMode(BUTTON_2, OUTPUT);
	  pinMode(BUTTON_3, OUTPUT);
	  pinMode(BUTTON_4, OUTPUT);

	  digitalWrite(BUTTON_1, NPN_OFF);
	  digitalWrite(BUTTON_2, NPN_OFF);
	  digitalWrite(BUTTON_3, PNP_OFF);
	  digitalWrite(BUTTON_4, PNP_OFF);
		
	  /* Enable Internal Pull-Up resistor */
	  digitalWrite (LED_PIC_UNLOCK, HIGH);

	  if(Serial)
	  {
		Serial.println("Waiting 20 seconds for PIC to complete initialization...");
	  }

	  /* Delay for 20 seconds to allow PIC target to complete initialization */
	  delay(20000);

	  if(Serial)
	  {
		Serial.println("Finished Waiting...");
	  }

	  if(Serial)
	  {
		Serial.println("Setup Complete...");
	  }

	  /* Use an interrupt to sense the LED connected to PIC12F1572 pin 3 toggling. */
	  attachInterrupt(digitalPinToInterrupt(LED_PIC_UNLOCK), pic_led_toggled, FALLING );

	  if(Serial)
	  {
		Serial.println("Setup Complete...");
	  }

	}

	void loop() {
	  
	  bool locked = true;
	  while(true == locked)
	  {
		/* Reset the toggle count */
		toggle_count = 0;

		/* Create a key */
		make_key(sequence, number);

		/* Output the combination to the serial port */
		printCombination(sequence, number);

		/* Push the buttons */
		apply_key(sequence, sizeof(sequence));
		
		/* Wait for the LED to stop blinking */
		delay(11000);

		Serial.print("Toggle Count:");
		Serial.println(toggle_count);

		/* Check for the unlock */
		if((toggle_count >= 100) && (toggle_count <= 120))
		{
		  /* Increment the number to make the next key */
		  number++;
		}
		else
		{
		  /* PIC must be unlocked */
		  locked = false;
		}
	  }

	  /* We think the device is unlocked */
	  while(1)
	  {
		printCombination(sequence, number);
	  }

	}
