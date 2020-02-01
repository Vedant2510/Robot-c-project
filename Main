#include "PC_FileIO.c"
#include "letters.c"

const char* fileName = "test.txt";

const float PAGE_WIDTH = 8.7; //cm
const float SCALE = 0.15; //1 unit is equal to 0.15 cm
const int SMALL = 1; //small space between letters
const int LARGE = 4; //large space between words
const int NEW = 25; //space between lines
const int TOL = 15; //robot can be up to 5deg skew
const int WRITE_POWER = 5; //5; //writing speed
const float BIG_RADIUS = 2.75; //side wheels
const float SMALL_RADIUS = 0.5; //arm wheel

const tMotor Y_LEFT = motorD;
const tMotor Y_RIGHT = motorA;
const tMotor X_MOTOR = motorB;
const tMotor Z_MOTOR = motorC;
const tSensors COLOR = S1;
const tSensors GYRO = S4;
const tSensors TOUCH = S3;

int GetIndex(char letter) //returns index of letter in the array
{
		if (letter == '.')
			return 26;
		else if (letter == '-')
			return 27;
		else if (letter >= 'a' && letter <= 'z') //lowercase
			return letter - 'a';
		else if (letter >= 'A' && letter <= 'Z')
			return letter - 'A';
		else
			return -1;
}

bool OnPaper() //checks to make sure the robot is on paper
{
	return SensorValue(COLOR) == (int)colorWhite;
}

bool NotHit() //checks to make sure the robot didn't hit anything
{
	return SensorValue(TOUCH) == 0;
}

float SpaceLeft(float& cur) //returns the space left in a line in cm
{
	return PAGE_WIDTH - cur;
}

float GetWidth(char toWrite[20]) // gets the width of a word in cm
{
	float width = 0;

	for (int curChar = 0; curChar < 20; curChar++)
	{
		int charIndex = GetIndex(toWrite[curChar]);

		if (charIndex != -1)
		{
			width += letters[charIndex].width;
			if (curChar != 20 -1)
			{width += SMALL;}
		}
	}

	return width*SCALE;
}

void MovePen(Point loc)//moves pen to a relative point
{
	while(!NotHit())
	{}

	int direction = 0;
	if (loc.x != 0)
	{
		int curEnc = nMotorEncoder[X_MOTOR];
		if (loc.x < 0)
		{direction = 1;}
		else
		{direction = -1;}

		float EncLimit = abs((loc.x*SCALE*180)/(PI*SMALL_RADIUS));

		motor[X_MOTOR] = direction*WRITE_POWER;
		while(abs(curEnc - nMotorEncoder[X_MOTOR]) < EncLimit)
		{}
		motor[X_MOTOR] = 0;
	}
	if (loc.y != 0)
	{
		int curEnc = nMotorEncoder[Y_LEFT];
		if (loc.y < 0)
		{direction = 1;}
		else
		{direction = -1;}

		float EncLimit = fabs((loc.y*SCALE*180)/(PI*BIG_RADIUS));

		motor[Y_LEFT] = motor[Y_RIGHT] = direction*WRITE_POWER;
		while(abs(curEnc - nMotorEncoder[Y_LEFT]) < EncLimit)
		{}
		motor[Y_LEFT] = motor[Y_RIGHT] = 0;
	}
}

void AddSmall() //adds a small space between letters
{
	Point small;
	small.x = SMALL;
	small.y = 0;
	MovePen(small);
}

void AddLarge() //adds a large space between words
{
	Point large;
	large.x = LARGE;
	large.y = 0;
	MovePen(large);
}

bool NotSkew() //checks to make sure the robot is aligned
{
	return abs(getGyroDegrees(GYRO)) < TOL;
}

void PauseTimer(int current, int& totalTime) //adds the current time to the total
{
	totalTime += current;
}

void PenUp() //lifts pen off the page
{
	nMotorEncoder[Z_MOTOR] = 0;
	motor[Z_MOTOR] = 30;

	wait1Msec(1000);

	motor[Z_MOTOR] = 0;
}

void PenDown() //puts the pen on the page
{
	nMotorEncoder[Z_MOTOR] = 0;
	motor[Z_MOTOR] = -30;

	wait1Msec(1000);
	motor[Z_MOTOR] = 0;
}

void Alert()
{
	setSoundVolume(50);
	playTone(695, 60);
}

void PressEnter() //waits for the user to press and release the enter button
{
	Alert();
	while(!getButtonPress(buttonEnter))
	{}
	while(getButtonPress(buttonEnter))
	{}
}

void ResetArm() //moves the arm all the way to the left
{
	motor[X_MOTOR] = 10;
	while (abs(nMotorEncoder[X_MOTOR]) > 0)
	{}
	wait1Msec(500);

	motor[X_MOTOR] = 0;
}

void NewLine(float& cur) //resets the arm and moves down a line
{
	ResetArm();
	Point new;
	new.x = 0;
	new.y = -NEW;
	MovePen(new);
	cur = 0;
}

void RefillPaper(float& cur, int& totalTime) //handles event when the robot reaches the end of the page
{
	PauseTimer(time1[T1], totalTime);
	do
	{
		eraseDisplay();
		displayString(1, "Place on a new piece of paper and press enter");
		PressEnter();
	} while(!OnPaper());

	eraseDisplay();
	ResetArm();
	cur = 0;
	time1[T1] = 0;
}

void WriteLetter(char letter) //follows the path for a letter
{
	int charIndex = GetIndex(letter);

	if (charIndex != -1)
	{
		MovePen(letters[charIndex].points[0]);
		PenDown();

		for (int curPoint = 1; curPoint < letters[charIndex].arrLength-1; curPoint++)
		{
			MovePen(letters[charIndex].points[curPoint]);
			wait1Msec(300);
		}

		PenUp();
		MovePen(letters[charIndex].points[letters[charIndex].arrLength-1]);
	}
}

bool TooLong(char toWrite[20]) //checks to see if the word needs to be split
{
	return GetWidth(toWrite) > PAGE_WIDTH;
}

int WriteLongWord(float& cur, char toWrite[20]) //adds hyphens where needed while writing a word
{
	int wordLength = 0;
	for (int character = 0; character < 19; character++)
	{
		if (GetIndex(toWrite[character]) != -1 && GetIndex(toWrite[character+1]) == -1)
		{wordLength = character+1;}
	}

	int length = 0;
	for (int curChar = 0; curChar < wordLength; curChar++)
	{
		int charIndex = GetIndex(toWrite[curChar]);

		float width = (SMALL + letters[charIndex].width + letters[27].width)*SCALE;

		if (SpaceLeft(cur) < width && curChar != wordLength-1)
		{
			WriteLetter('-');

			if (!OnPaper())
				{RefillPaper(cur), totalTime;}
			else
				{NewLine(cur);}

			WriteLetter(toWrite[curChar]);
			AddSmall();
			length += 2;
		}
		else
		{
			WriteLetter(toWrite[curChar]);
			if (curChar != wordLength-1)
				{AddSmall();}
			length++;
		}

		cur += width;
	}

	return length;
}

int WriteWord(char toWrite[20]) //writes a word
{
	int wordLength = 0;
	for (int character = 0; character < 19; character++)
	{
		if (GetIndex(toWrite[character]) != -1 && GetIndex(toWrite[character+1]) == -1)
		{wordLength = character+1;}
	}

	for (int curChar = 0; curChar < wordLength; curChar++)
	{
		WriteLetter(toWrite[curChar]);
		if (curChar != 19)
		{AddSmall();}
	}
	AddLarge();

	return wordLength;
}

void getToWrite(char* strWord, char toWrite[20])
{
	for (int i  = 0; i < 20; i++) //resetting array
	{
		toWrite[i] = '&';
	}

	for (int i = 0; i < strlen(strWord); i++)
	{
		toWrite[i] = strWord[i];
	}
}

task main()
{
		TFileHandle fin;
		bool fileOkay = openReadPC(fin, fileName);

		if (fileOkay)
		{
		SensorType[TOUCH] = sensorEV3_Touch;
		SensorType[GYRO] = sensorEV3_Gyro;
		SensorType[COLOR] = sensorEV3_Color;
		wait1Msec(50);
		SensorMode[GYRO] = modeEV3Gyro_RateAndAngle;
		SensorMode[COLOR] = modeEV3Color_Color;
		wait1Msec(50);

		do
		{
			displayString(1, "Place on a piece of paper");
			displayString(2, "and press enter");
			PressEnter();
		} while(!OnPaper());
		eraseDisplay();
		displayString(1, "Printing in progress");

		nMotorEncoder[Y_LEFT] = 0;
		nMotorEncoder[X_MOTOR] = 0;
		resetGyro(GYRO);
		int totalTime = 0;
		int totalChar = 0;
		getArr();
		PenUp();
		float cur = 0;
		time1[T1] = 0;
		char toWrite[20];

	    string strWord = "";
		while (readTextPC(fin, strWord))
		{
			getToWrite(strWord, toWrite);
			if (!NotSkew())
			{
				PauseTimer(time1[T1], totalTime);
				do
				{
					eraseDisplay();
					displayString(1, "Align robot on paper");
					displayString(2, "and press enter");
					PressEnter();
				} while (!OnPaper());
				resetGyro(GYRO);
				eraseDisplay();
				displayString(1, "Printing in progress");
			}

			float spaceleft = SpaceLeft(cur);
			float width = GetWidth(toWrite);

			displayString(1, "Printing in progress");

			if (width > spaceleft && !TooLong(toWrite)) //not enough space left on the current line
			{
				if (!OnPaper())
				{RefillPaper(cur, totalTime);}
				else
				{NewLine(cur);}
			}

			if (TooLong(toWrite))
			{totalChar += WriteLongWord(cur, toWrite);}
			else
			{totalChar += WriteWord(toWrite);}

			cur += width + LARGE;
		}

			PauseTimer(time1[T1], totalTime);
			ResetArm();

			eraseDisplay();
			displayString(1, "Printing complete");
			displayString(2, "Time: %f", totalTime/1000.0);
			displayString(3, "Characters: %d", totalChar);

			closeFilePC(fin);

		}
		else
		{
			displayString(1, "Problem opening file.");
			displayString(2, "Press enter to end program");
		}
		PressEnter();
}
