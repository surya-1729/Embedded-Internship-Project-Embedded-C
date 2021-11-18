*******************************************************************************************************
*        name:                    Surya VaraPrasad Alla                                               *
*        matriculation number:    1536227                                                             *
*        e-mail:                  surya.alla@student.uni-siegen.de                                    *
*******************************************************************************************************
*        Hardware Setup                                                                               *
*                                                                                                     *
*             VCC               MSP430FR5969                                                          *
*              |             -----------------                                                        *
*           (SENSOR)        |                 |                                                       *
*              +-------- -->|P4.2/A10     P4.6|--> (LED1)                                             *
*              |            |             P1.0|--> (LED2)                                             *
*             |R|         ->|S1 P4.5          |                                                       *
*                         ->|S2 P1.1          |                                                       *
                            |                 |
*              |             -----------------                                                        *
*             GND                                                                                     *
*******************************************************************************************************


#include <msp430fr5969.h>
#include <stdint.h>

#define THRS 4000

volatile uint16_t adc_val = 0;
uint8_t i = 0;
uint8_t x = 0;
uint8_t s = 0;
uint8_t h = 0;


// states of ADC
volatile struct
{
    uint8_t Adc : 2;                    // Current ADC state:
                                        // 0:   Uninitialized after reset.
                                        // 1:   Transition to fast sampling.
                                        // 2:   Fast sampling.
} System;

//  setting up fast sampling
typedef enum                            // State values of ADC FSM:
{
    ADC_RST,                            // 0:   Uninitialized after reset.
    ADC_T_FAST,                         // 1:   Transition to fast sampling.
    ADC_FAST,                           // 2:   Fast sampling.
} SYSTEM_ADC;

// MAIN FSM
volatile struct
{
    uint8_t MAIN : 0;                    // MAIN ROUTINE
                                         // 0:MAIN_SLEEP
                                         // 1:MAIN_WAKEUP
                                         // 2:MAIN_MEASURE
                                         // 3:MAIN_TRANSMIT
                                         // 4:MAIN_ERROR
}States;
typedef enum                             // State values of main FSM:
{
    MAIN_SLEEP,
    MAIN_WAKEUP,
    MAIN_MEASURE,
    MAIN_TRANSMIT,
    MAIN_ERROR,
}


void main(void)
{
    // Stop watchdog timer.
    WDTCTL = WDTPW | WDTHOLD;


    // Initialize the clock system to generate 1 MHz DCO clock.
    CSCTL0_H    = CSKEY_H;              // Unlock CS registers.
    CSCTL1      = DCOFSEL_0;            // Set DCO to 1 MHz, DCORSEL for
                                        //   high speed mode not enabled.
    CSCTL2      = SELA__VLOCLK |        // Set ACLK = VLOCLK = 10 kHz.
                  SELS__DCOCLK |        //   Set SMCLK = DCOCLK.
                  SELM__DCOCLK;         //   Set MCLK = DCOCLK.
                                        // SMCLK = MCLK = DCOCLK = 1 MHz.
    CSCTL3      = DIVA__1 |             //   Set ACLK divider to 1.
                  DIVS__1 |             //   Set SMCLK divider to 1.
                  DIVM__1;              //   Set MCLK divider to 1.
                                        // Set all dividers to 1.
    CSCTL0_H    = 0;                    // Lock CS registers.


    // Initialize unused GPIOs to minimize energy-consumption.
    // Port 1:
    P1DIR = 0xFF;
    P1OUT = 0x00;
    // Port 2:
    P2DIR = 0xFF;
    P2OUT = 0x00;
    // Port 3:
    P3DIR = 0xFF;
    P3OUT = 0x00;
    // Port 4:
    P4DIR = 0xFF;
    P4OUT = 0x00;
    // Port J:
    PJDIR = 0xFFFF;
    PJOUT = 0x0000;

    // Initialize port 1:
    P1DIR |= BIT0;                      // P1.0 - output for LED2, off.
    P1OUT &= ~BIT0;

    // Initialize port 4:
    P4DIR |= BIT6;                      // P4.6 - output for LED1, off.
    P4OUT &= ~BIT6;

    // Initialize ADC input pins:

    P4SEL1 |= BIT2;              // select function
    P4SEL0 |= BIT2;

    // Disable the GPIO power-on default high-impedance mode to activate the
    // previously configured port settings.
    PM5CTL0 &= ~LOCKLPM5;

    /* Initialization of the reference voltage generator */

    // Configure the internal voltage reference for 2.5 V:

    REFCTL0 |= REFVSEL1;      // selector reference voltage - 2.5V
    REFCTL0 &= ~REFOUT;       // Reference output Buffer On
    REFCTL0 |= REFON;         // Reference On

    // Initialization of the system variables.
    System.Adc = ADC_RST;

    // Enable interrupts globally.
    __bis_SR_register(GIE);



  switch(States.MAIN)
    {
    case MAIN_SLEEP:

      //  switch case based on state of sensor
      switch(System.Adc)
      {
      case ADC_RST:                         // Initialize ADC right after reset.
        System.Adc = ADC_T_FAST;
        break;

      case ADC_T_FAST:                      // Initialize ADC to sample fast.
        // Configure the analog-to-digital converter such that it samples
        // as fast as possible.
        ADC12CTL0 = 0;                      // Disable ADC trigger.
        ADC12CTL0 |= ADC12ON;               // Turn on ADC.
        ADC12CTL0 |= ADC12MSC;              // Multiple sample and conversion mode.
        ADC12CTL0 |= ADC12SHT0_6;           // Sample-and-hold time of 128 cycles of MODCLK.
        ADC12CTL1  = 0;
        ADC12CTL1 |= ADC12SHP;              // Use sampling timer.
        ADC12CTL1 |= ADC12CONSEQ_2;         // Repeat-single-channel mode.
        ADC12CTL2  = 0;
        ADC12CTL2 |= ADC12RES_0;            // 8 bit conversion resolution.
        ADC12IER0  |= ADC12IE0;             // Enable ADC conversion interrupt.
        ADC12MCTL0 |= ADC12INCH_10;         // Select ADC input channel A10.
        ADC12MCTL0 |= ADC12VRSEL_1;         // Reference voltages V+ = VREF buffered
                                            // = 2.5V and V- = AVSS = GND.
        ADC12CTL0  |= ADC12ENC | ADC12SC;   // Enable ADC trigger.
                                            // Trigger sampling.
        while(ADC12IFGR0 & ADC12IFG10);     // Wait for first 12 bit sample.
        // Go to the fast sampling process.
        System.Adc = ADC_FAST;
        break;

      case ADC_FAST:                        // Process using fast sampling.

        // check threshold
        if (adc_val > THRS)
        {
         __delay_cycles(4000000);
         if (adc_val > THRS)
         {
           States.MAIN = MAIN_WAKEUP;
         }
         else if(adc_val == 0)
         {
             States.MAIN = MAIN_ERROR;
         }
         else
         {
         States.MAIN = MAIN_SLEEP;
         }
        }
        break;
      }

    case MAIN_WAKEUP:

      //  preamble collection from sensor digital inputs
      for (i = 0; i < 8 ; i++)
      {
        if (adc_val > THRS)
        {
            x |= 1 << (7-i);
            __delay_cycles(500000);
        }
        else
        {
            x |= 0 << (7-i);
            __delay_cycles(500000);
        }
      }
      //  preamble check with user defined byte value
      if (x == 0b01101010 && adc_val > THRS)
      {
         P1OUT |= BIT0; // acknowledgement to CARL
         __delay_cycles(250000);
         States.MAIN = MAIN_MEASURE;
      }
      else if(adc_val == 0)
      {
          States.MAIN = MAIN_ERROR;
      }
      else
      {
         States.MAIN = MAIN_SLEEP;
      }
    break;

    case MAIN_MEASURE:

        if(adc_val > THRS)
        {
        // measurements 8 stress and 8 humidity
        P4OUT |= BIT6;  // LED1 ON for 10ms
        __delay_cycles(10000);
        for (i = 0; i < 8 ; i++)
        {
            __delay_cycles(250000);
            if (!(P4IN & BIT5))
            {
              s |= 1 << (7-i);
             __delay_cycles(475000);
            }
            else
            {
              s |= 0 << (7-i);
              _delay_cycles(475000);
            }
        }
        P4OUT |= BIT6;  // LED1 ON for 10ms
        __delay_cycles(10000);
        for (i = 0; i < 8 ; i++)
        {
            __delay_cycles(250000);
             if (!(P1IN & BIT1))
             {
               h |= 1 << (7-i);
               _delay 475ms;
             }
             else
             {
               h |= 0 << (7-i);
               _delay 475ms;
             }
        }
        States.MAIN = MAIN_TRANSMIT;
        }
        else if(adc_val == 0)
        {
            States.MAIN = MAIN_ERROR;
        }

        else
        {
          States.MAIN = MAIN_SLEEP;
        }


    case MAIN_TRANSMIT:

       if(adc_val > 4000)
       {
       // transmit preamble (0b10010101 MSB)
        uint8_t p = 0b10010101;
       for (i = 0; i < 8; i++)
       {
           uint8_t b = (p & ( 1 << (7-i) )) >> (7-i); // bit value in preamble
           if (b == 1)
           {
               P1OUT |= BIT0; // LED2 on 75% duty cycle
               _delay 0.75*0.5sec;
           }
           else
           {
               P1OUT |= BIT0;  // LED2 on 25% duty cycle
               _delay 0.25*0.5sec;
           }
       }

       // transmit stress and humidity measurements
       for (i = 0; i < 8; i++)
       {
           uint8_t b = (s & ( 1 << (7-i) )) >> (7-i); // bit value in stress s
           if (b == 1)
           {
               P1OUT |= BIT0; // LED2 on 75% duty cycle
               _delay 0.75*0.5sec;
           }
           else
           {
               P1OUT |= BIT0;  // LED2 on 25% duty cycle
               _delay 0.25*0.5sec;
           }

       }
       // HUMIDITY TRANSMIT
       for (i = 0; i < 8; i++)
       {
           uint8_t b = (h & ( 1 << (7-i) )) >> 7-i; // bit value in humidity h
           if (b == 1)
           {
               P1OUT |= BIT0; // LED2 on 75% duty cycle
               _delay 0.75*0.5sec;
           }
           else
           {
               P1OUT |= BIT0;  // LED2 on 25% duty cycle
               _delay 0.25*0.5sec;
           }

       }
       States.MAIN = MAIN_SLEEP;
       }
       else if(adc_val == 0)
       {
           States.MAIN = MAIN_ERROR;
       }
       else
       {
         States.MAIN = MAIN_SLEEP;
       }

    case MAIN_ERROR:
        P1OUT |= BIT0;
        _delay 1000ms;
        States.MAIN = MAIN_SLEEP;

  }

}

    /* ISR ADC */
    #pragma vector = ADC12_VECTOR
    __interrupt void ADC12_ISR(void)
    {
        switch(__even_in_range(ADC12IV, ADC12IV_ADC12RDYIFG))
        {
        case ADC12IV_NONE:        break;        // Vector  0:  No interrupt
        case ADC12IV_ADC12OVIFG:  break;        // Vector  2:  ADC12MEMx Overflow
        case ADC12IV_ADC12TOVIFG: break;        // Vector  4:  Conversion time overflow
        case ADC12IV_ADC12HIIFG:  break;        // Vector  6:  ADC12BHI
        case ADC12IV_ADC12LOIFG:  break;        // Vector  8:  ADC12BLO
        case ADC12IV_ADC12INIFG:  break;        // Vector 10:  ADC12BIN
        case ADC12IV_ADC12IFG0:                 // Vector 12:  ADC12MEM0 Interrupt

            adc_val = ADC12MEM0;

            break;
        case ADC12IV_ADC12IFG1:   break;        // Vector 14:  ADC12MEM1
        case ADC12IV_ADC12IFG2:   break;        // Vector 16:  ADC12MEM2
        case ADC12IV_ADC12IFG3:   break;        // Vector 18:  ADC12MEM3
        case ADC12IV_ADC12IFG4:   break;        // Vector 20:  ADC12MEM4
        case ADC12IV_ADC12IFG5:   break;        // Vector 22:  ADC12MEM5
        case ADC12IV_ADC12IFG6:   break;        // Vector 24:  ADC12MEM6
        case ADC12IV_ADC12IFG7:   break;        // Vector 26:  ADC12MEM7
        case ADC12IV_ADC12IFG8:   break;        // Vector 28:  ADC12MEM8
        case ADC12IV_ADC12IFG9:   break;        // Vector 30:  ADC12MEM9
        case ADC12IV_ADC12IFG10:  break;        // Vector 32:  ADC12MEM10
        case ADC12IV_ADC12IFG11:  break;        // Vector 34:  ADC12MEM11
        case ADC12IV_ADC12IFG12:  break;        // Vector 36:  ADC12MEM12
        case ADC12IV_ADC12IFG13:  break;        // Vector 38:  ADC12MEM13
        case ADC12IV_ADC12IFG14:  break;        // Vector 40:  ADC12MEM14
        case ADC12IV_ADC12IFG15:  break;        // Vector 42:  ADC12MEM15
        case ADC12IV_ADC12IFG16:  break;        // Vector 44:  ADC12MEM16
        case ADC12IV_ADC12IFG17:  break;        // Vector 46:  ADC12MEM17
        case ADC12IV_ADC12IFG18:  break;        // Vector 48:  ADC12MEM18
        case ADC12IV_ADC12IFG19:  break;        // Vector 50:  ADC12MEM19
        case ADC12IV_ADC12IFG20:  break;        // Vector 52:  ADC12MEM20
        case ADC12IV_ADC12IFG21:  break;        // Vector 54:  ADC12MEM21
        case ADC12IV_ADC12IFG22:  break;        // Vector 56:  ADC12MEM22
        case ADC12IV_ADC12IFG23:  break;        // Vector 58:  ADC12MEM23
        case ADC12IV_ADC12IFG24:  break;        // Vector 60:  ADC12MEM24
        case ADC12IV_ADC12IFG25:  break;        // Vector 62:  ADC12MEM25
        case ADC12IV_ADC12IFG26:  break;        // Vector 64:  ADC12MEM26
        case ADC12IV_ADC12IFG27:  break;        // Vector 66:  ADC12MEM27
        case ADC12IV_ADC12IFG28:  break;        // Vector 68:  ADC12MEM28
        case ADC12IV_ADC12IFG29:  break;        // Vector 70:  ADC12MEM29
        case ADC12IV_ADC12IFG30:  break;        // Vector 72:  ADC12MEM30
        case ADC12IV_ADC12IFG31:  break;        // Vector 74:  ADC12MEM31
        case ADC12IV_ADC12RDYIFG: break;        // Vector 76:  ADC12RDY
        default: break;
        }
    }
