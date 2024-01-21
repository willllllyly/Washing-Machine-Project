#include "mbed.h"
#include <chrono>
#include <cstdio>

#define NOTE_C4  262 
#define NOTE_D4  294
#define NOTE_F4  349

DigitalOut rled(PC_0);
DigitalOut gled1(PC_1); // Green led closest to the red
DigitalOut gled2(PB_0);
DigitalOut gled3(PA_4);
DigitalIn button1(PC_10); // Left button
DigitalIn button2(PC_11); // Middle button
DigitalIn button3(PD_2); // Right button
BusOut sevenseg(PB_12,PA_12, PB_1, PB_15, PB_14, PA_11, PB_11); // Bus out to control the seven seg display
BusOut rgb(PB_3,PB_5,PB_4); // Bus out to control the colour of the rgb LED
AnalogIn pot1(PA_6);
AnalogIn ldr(PC_2);
PwmOut buzzer(PA_15);
PwmOut g_rgb(PB_5); // Specific output setup for the green leg of the rgb LED

int cycle = 0; // Set variable to 0 
const int C_major_scale[] = {NOTE_C4, NOTE_C4, NOTE_F4, NOTE_D4,NOTE_F4}; // Array with the notes in their required order
bool check = false; // Make check variables false to begin
bool check2 = false;
void wm_on();
//void wm_off1();
void wm_off2();
void ldr_disp(float current_ldr);
void countdown();
void play_note(int frequency);
int HexDec0 = {0x00}; // Off
int HexDec1 = {0x73}; // Perm press
int HexDec2 = {0x57}; // High Heat 
int HexDec3 = {0x99}; // Low Heat
int HexDec_3 = {0x6e}; // Number 3
int HexDec_2 = {0x7a}; // Number 2
int HexDec_1 = {0x06}; // Number 1
int HexDec_0 = {0x3f}; // Number 0
float pot_val = 0.0f; // Set empty float value for pot
float ldr_val = 0.0f; // Set empty flaot value for ldr

int main()
{
    printf("If Red LED is lit, washing machine is OFF\n");
    while (1) { // Infinite loop to always check if button is pressed
        if (check == false) // What the code intitially carries out as it's set to false already
            gled1.write(0);
            rled.write(1);
            if (button1.read() == 1){
                check = !check; //Inverts bool value everytime you press the button
                wm_on(); // Carry out the wm_on function   
            if (check == true) { // If button 1 is pressed, then carries out this set of statements 
                while(check == true){ // Loop to always be checking what the select dial (pot) is on 
                pot_val = pot1.read()*3.3; // To make it easier to split into sections
                     if (button3.read() == 1){ // If button 3 is pressed carry out one of these statements
                        cycle = sevenseg.read(); // Reads what the hex number currently displayed on the seven seg
                        check2 = !check2;
                        if (cycle == (HexDec1)){ //
                            gled2.write(1);
                            printf("You have selected Perm Press\n");
                            ThisThread::sleep_for(500ms);
                            countdown();
                        }
                        else if (cycle == (HexDec2)){
                            gled2.write(1);
                            printf("You have selected High Heat Wash\n");
                            ThisThread::sleep_for(500ms);
                            countdown();
                        }
                        else
                        {
                            gled2.write(1);
                            printf("You Have selected Low Heat Wash\n");
                            ThisThread::sleep_for(500ms);
                            countdown();
                        }
                    }
                    if (pot_val <= 0.3) // Depending on the pot value, display a certain letter and thus wash cycle
                    {

                    }
                    else if (pot_val > 0.3 && pot_val <=1.3) {
                    sevenseg.write(int (HexDec1));
                    //printf("Perm Press");
                    }
                    else if (pot_val > 1.3 && pot_val <=2.3) {
                    sevenseg.write(int (HexDec2));
                    //printf("High Heat Wash");
                    }
                    else if (pot_val > 2.3 && pot_val <=3.3) {
                    sevenseg.write(int (HexDec3));
                    //printf("Low Heat Wash");
                    }
                    if (button2.read() == 1){ // If button 2 is  pressed then turn off the machine
                       wm_off2();
                    }
                }
            }
            }

    }
}

void wm_on() {
    gled1.write(1); // Turn the first green LED on, turn all others off
    rled.write(0);
    gled2.write(0);
    printf("\nOn\n");
    ThisThread::sleep_for(500ms);
    printf("Please select your wash cycle: \n");
}
//void wm_off1() {
 //    gled1.write(0);
   //  rled.write(1);
     //printf("Off\n");
     //ThisThread::sleep_for(500ms);
//}
void wm_off2() {
     gled1.write(0); // Turn all other LEDs and Displays off apart from the red LED
     rled.write(1);
     gled2.write(0);
     gled3.write(0);
     g_rgb.write(0);
     rgb.write(int (0x00));
     buzzer.write(0.0);
     check = false; // Turn checks back to false to stop other loops running
     check2 = false;
     ThisThread::sleep_for(500ms);
     sevenseg.write(int (HexDec0));
     printf("Washing machine is OFF\n");
     ThisThread::sleep_for(500ms);

}
void ldr_disp(float current_ldr) {
    if (ldr_val <= 60.0){ // If the washing door is "Closed" after cycle is complete, carry out statements,
                          // letting the user know the cycle has finished; Turn rgb led green and play buzzer
        rgb.write(int (0x00));
        g_rgb.write(1);
        ThisThread::sleep_for(800ms);
        for(int i = 0; i < 5; i++){ // For loop to play each note in the array
            play_note(C_major_scale[i]); // Carry out play note function
        }
    }
    else { // If a light is shone on the ldr "Door open", Then turn everything off and show a blue light
        buzzer.write(0.0);
        g_rgb.write(0);
        rgb.write(int (0x6e));
    }
}
void play_note(int frequency){
    buzzer.period_us((float) 1000000.0f/ (float) frequency); // Set the period of the pwm signal (in us)
    buzzer.pulsewidth_us(buzzer.read_period_us()/2); // Set the pulse width of the pwm to 0.5x the period
    ThisThread::sleep_for(350ms); // PLay the each note for 350ms
}
void countdown() {
    sevenseg.write(int (HexDec_3)); // Have a countdown timer displaying from 3 to 0
    ThisThread::sleep_for(1s);
    sevenseg.write(int (HexDec_2));
    ThisThread::sleep_for(1s);
    sevenseg.write(int (HexDec_1));
    ThisThread::sleep_for(1s);
    sevenseg.write(int (HexDec_0));
    printf("\nWashing Cycle is Complete!!\n");
    ThisThread::sleep_for(500ms);
    gled3.write(1); // Turn on the final green LED
    ThisThread::sleep_for(1s);
    while (check2 == true) { // While loop to constantly be chaecking the ldr value and whether the door is open or shut
        ldr_val = ldr.read()*100; // Get the ldr reading as a percentage
        ldr_disp(ldr_val); // Carry out the ldr function
        if (button2.read() == 1){ // If button 2 is pressed then turn it off
            wm_off2();
        
        }
        check2 = false;

    }

}
