---
layout: post
title: pwnable blackjack
category: [pwnable, security]
tags: [pwnable, Toddler's Bottle]
fullview: true
comments: true
use_math: true
author: fault2000
---

블랙잭을 플레이할 수 있는 프로그램을 실행할 수 있는 포트만을 준다. 접속해보자.

```c
// Programmer: Vladislav Shulman
// Final Project
// Blackjack
 
// Feel free to use any and all parts of this program and claim it as your own work
 
//FINAL DRAFT
 
#include <stdlib.h>
#include <stdio.h>
#include <math.h>
#include <time.h>                //Used for srand((unsigned) time(NULL)) command
#include <process.h>             //Used for system("cls") command
 
#define spade 06                 //Used to print spade symbol
#define club 05                  //Used to print club symbol
#define diamond 04               //Used to print diamond symbol
#define heart 03                 //Used to print heart symbol
#define RESULTS "Blackjack.txt"  //File name is Blackjack
 
//Global Variables
int k;
int l;
int d;
int won;
int loss;
int cash = 500;
int bet;
int random_card;
int player_total=0;
int dealer_total;
 
//Function Prototypes
int clubcard();      //Displays Club Card Image
int diamondcard();   //Displays Diamond Card Image
int heartcard();     //Displays Heart Card Image
int spadecard();     //Displays Spade Card Image
int randcard();      //Generates random card
int betting();       //Asks user amount to bet
void asktitle();     //Asks user to continue
void rules();        //Prints "Rules of Vlad's Blackjack" menu
void play();         //Plays game
void dealer();       //Function to play for dealer AI
void stay();         //Function for when user selects 'Stay'
void cash_test();    //Test for if user has cash remaining in purse
void askover();      //Asks if user wants to continue playing
void fileresults();  //Prints results into Blackjack.txt file in program directory
 
//Main Function
int main(void)
{
    int choice1;
    printf("\n");
    printf("\n");
    printf("\n");
    printf("\n              222                111                            ");
    printf("\n            222 222            11111                              ");
    printf("\n           222   222          11 111                            "); 
    printf("\n                222              111                               "); 
    printf("\n               222               111                           ");   
    printf("\n");
    printf("\n%c%c%c%c%c     %c%c            %c%c         %c%c%c%c%c    %c    %c                ", club, club, club, club, club, spade, spade, diamond, diamond, heart, heart, heart, heart, heart, club, club);  
    printf("\n%c    %c    %c%c           %c  %c       %c     %c   %c   %c              ", club, club, spade, spade, diamond, diamond, heart, heart, club, club);            
    printf("\n%c    %c    %c%c          %c    %c     %c          %c  %c               ", club, club, spade, spade, diamond, diamond, heart, club, club);                        
    printf("\n%c%c%c%c%c     %c%c          %c %c%c %c     %c          %c %c              ", club, club, club, club, club, spade, spade, diamond, diamond, diamond, diamond, heart, club, club);      
    printf("\n%c    %c    %c%c         %c %c%c%c%c %c    %c          %c%c %c             ", club, club, spade, spade, diamond, diamond, diamond, diamond, diamond, diamond, heart, club, club, club);                       
    printf("\n%c     %c   %c%c         %c      %c    %c          %c   %c               ", club, club, spade, spade, diamond, diamond, heart, club, club);                                         
    printf("\n%c     %c   %c%c        %c        %c    %c     %c   %c    %c             ", club, club, spade, spade, diamond, diamond, heart, heart, club, club);                                                            
    printf("\n%c%c%c%c%c%c    %c%c%c%c%c%c%c   %c        %c     %c%c%c%c%c    %c     %c            ", club, club, club, club, club, club, spade, spade, spade, spade, spade, spade, spade, diamond, diamond, heart, heart, heart, heart, heart, club, club);                                                                                     
    printf("\n");     
    printf("\n                        21                                   ");
     
    printf("\n     %c%c%c%c%c%c%c%c      %c%c         %c%c%c%c%c    %c    %c                ", diamond, diamond, diamond, diamond, diamond, diamond, diamond, diamond, heart, heart, club, club, club, club, club, spade, spade);                     
    printf("\n        %c%c        %c  %c       %c     %c   %c   %c              ", diamond, diamond, heart, heart, club, club, spade, spade);                                      
    printf("\n        %c%c       %c    %c     %c          %c  %c               ", diamond, diamond, heart, heart, club, spade, spade);                                           
    printf("\n        %c%c       %c %c%c %c     %c          %c %c              ", diamond, diamond, heart, heart, heart, heart, club, spade, spade);                                     
    printf("\n        %c%c      %c %c%c%c%c %c    %c          %c%c %c             ", diamond, diamond, heart, heart, heart, heart, heart, heart, club, spade, spade, spade);                                                
    printf("\n        %c%c      %c      %c    %c          %c   %c               ", diamond, diamond, heart, heart, club, spade, spade);                                                                               
    printf("\n     %c  %c%c     %c        %c    %c     %c   %c    %c             ", diamond, diamond, diamond, heart, heart, club, spade, spade);                                                                                                               
    printf("\n      %c%c%c      %c        %c     %c%c%c%c%c    %c     %c            ", diamond, diamond, diamond, heart, heart, club, club, club, club, club, spade, spade);                                                                                                                                        
    printf("\n");  
    printf("\n         222                     111                         ");
    printf("\n        222                      111                         ");
    printf("\n       222                       111                         ");
    printf("\n      222222222222222      111111111111111                       ");
    printf("\n      2222222222222222    11111111111111111                         ");
    printf("\n");
    printf("\n");
     
    asktitle();
     
    printf("\n");
    printf("\n");
    system("pause");
    return(0);
} //end program
 
void asktitle() // Function for asking player if they want to continue
{
    char choice1;
    int choice2;
     
     printf("\n                 Are You Ready?");
     printf("\n                ----------------");
     printf("\n                      (Y/N)\n                        ");
     scanf("\n%c",&choice1);
 
    while((choice1!='Y') && (choice1!='y') && (choice1!='N') && (choice1!='n')) // If invalid choice entered
    {                                                                           
        printf("\n");
        printf("Incorrect Choice. Please Enter Y for Yes or N for No.\n");
        scanf("%c",&choice1);
    }
 
 
    if((choice1 == 'Y') || (choice1 == 'y')) // If yes, continue. Prints menu.
    { 
            system("cls");
            printf("\nEnter 1 to Begin the Greatest Game Ever Played.");
            printf("\nEnter 2 to See a Complete Listing of Rules.");
            printf("\nEnter 3 to Exit Game. (Not Recommended)");
            printf("\nChoice: ");
            scanf("%d", &choice2); // Prompts user for choice
            if((choice2<1) || (choice2>3)) // If invalid choice entered
            {
                printf("\nIncorrect Choice. Please enter 1, 2 or 3\n");
                scanf("%d", &choice2);
            }
            switch(choice2) // Switch case for different choices
            {   
                case 1: // Case to begin game
                   system("cls");
                    
                   play();
                                       
                   break;
                    
                case 2: // Case to see rules
                   system("cls");
                   rules();
                   break;
                    
                case 3: // Case to exit game
                   printf("\nYour day could have been perfect.");
                   printf("\nHave an almost perfect day!\n\n");
                   system("pause");
                   exit(0);
                   break;
                    
                default:
                   printf("\nInvalid Input");
            } // End switch case
    } // End if loop
    
             
 
    else if((choice1 == 'N') || (choice1 == 'n')) // If no, exit program
    {
        printf("\nYour day could have been perfect.");
        printf("\nHave an almost perfect day!\n\n");
        system("pause");
        exit(0);
    }
     
    return;
} // End function
 
void rules() //Prints "Rules of Vlad's Blackjack" list
{
     char choice1;
     int choice2;
      
     printf("\n           RULES of VLAD's BLACKJACK");
     printf("\n          ---------------------------");
     printf("\nI.");
     printf("\n     Thou shalt not question the odds of this game.");
     printf("\n      %c This program generates cards at random.", spade);
     printf("\n      %c If you keep losing, you are very unlucky!\n", diamond);
      
     printf("\nII.");
     printf("\n     Each card has a value.");
     printf("\n      %c Number cards 1 to 10 hold a value of their number.", spade);
     printf("\n      %c J, Q, and K cards hold a value of 10.", diamond);
     printf("\n      %c Ace cards hold a value of 11", club);
     printf("\n     The goal of this game is to reach a card value total of 21.\n");
      
     printf("\nIII.");
     printf("\n     After the dealing of the first two cards, YOU must decide whether to HIT or STAY.");
     printf("\n      %c Staying will keep you safe, hitting will add a card.", spade);
     printf("\n     Because you are competing against the dealer, you must beat his hand.");
     printf("\n     BUT BEWARE!.");
     printf("\n      %c If your total goes over 21, you will LOSE!.", diamond);
     printf("\n     But the world is not over, because you can always play again.\n");
     printf("\n%c%c%c YOUR RESULTS ARE RECORDED AND FOUND IN SAME FOLDER AS PROGRAM %c%c%c\n", spade, heart, club, club, heart, spade);
     printf("\nWould you like to go the previous screen? (I will not take NO for an answer)");
     printf("\n                  (Y/N)\n                    ");
     scanf("\n%c",&choice1);
      
     while((choice1!='Y') && (choice1!='y') && (choice1!='N') && (choice1!='n')) // If invalid choice entered
    {                                                                           
        printf("\n");
        printf("Incorrect Choice. Please Enter Y for Yes or N for No.\n");
        scanf("%c",&choice1);
    }
 
 
    if((choice1 == 'Y') || (choice1 == 'y')) // If yes, continue. Prints menu.
    { 
            system("cls");
            asktitle();
    } // End if loop
    
             
 
    else if((choice1 == 'N') || (choice1 == 'n')) // If no, convinces user to enter yes
    {
        system("cls");
        printf("\n                 I told you so.\n");
        asktitle();
    }
     
    return;
} // End function
 
int clubcard() //Displays Club Card Image
{
     
     
    srand((unsigned) time(NULL)); //Generates random seed for rand() function
    k=rand()%13+1;
     
    if(k<=9) //If random number is 9 or less, print card with that number
    {
    //Club Card
    printf("-------\n");
    printf("|%c    |\n", club);
    printf("|  %d  |\n", k);
    printf("|    %c|\n", club);
    printf("-------\n");
    }
     
     
    if(k==10) //If random number is 10, print card with J (Jack) on face
    {
    //Club Card
    printf("-------\n");
    printf("|%c    |\n", club);
    printf("|  J  |\n");
    printf("|    %c|\n", club);
    printf("-------\n");
    }
     
     
    if(k==11) //If random number is 11, print card with A (Ace) on face
    {
    //Club Card
    printf("-------\n");
    printf("|%c    |\n", club);
    printf("|  A  |\n");
    printf("|    %c|\n", club);
    printf("-------\n");
    if(player_total<=10) //If random number is Ace, change value to 11 or 1 depending on dealer total
         {
             k=11;
         }
          
         else
         {
 
             k=1;
         }
    }
     
     
    if(k==12) //If random number is 12, print card with Q (Queen) on face
    {
    //Club Card
    printf("-------\n");
    printf("|%c    |\n", club);
    printf("|  Q  |\n");
    printf("|    %c|\n", club);
    printf("-------\n");
    k=10; //Set card value to 10
    }
     
     
    if(k==13) //If random number is 13, print card with K (King) on face
    {
    //Club Card
    printf("-------\n");
    printf("|%c    |\n", club);
    printf("|  K  |\n");
    printf("|    %c|\n", club);
    printf("-------\n");
    k=10; //Set card value to 10
    }
    return k;           
}// End function
 
int diamondcard() //Displays Diamond Card Image
{
     
     
    srand((unsigned) time(NULL)); //Generates random seed for rand() function
    k=rand()%13+1;
     
    if(k<=9) //If random number is 9 or less, print card with that number
    {
    //Diamond Card
    printf("-------\n");
    printf("|%c    |\n", diamond);
    printf("|  %d  |\n", k);
    printf("|    %c|\n", diamond);
    printf("-------\n");
    }
     
    if(k==10) //If random number is 10, print card with J (Jack) on face
    {
    //Diamond Card
    printf("-------\n");
    printf("|%c    |\n", diamond);
    printf("|  J  |\n");
    printf("|    %c|\n", diamond);
    printf("-------\n");
    }
     
    if(k==11) //If random number is 11, print card with A (Ace) on face
    {
    //Diamond Card
    printf("-------\n");
    printf("|%c    |\n", diamond);
    printf("|  A  |\n");
    printf("|    %c|\n", diamond);
    printf("-------\n");
    if(player_total<=10) //If random number is Ace, change value to 11 or 1 depending on dealer total
         {
             k=11;
         }
          
         else
         {
             k=1;
         }
    }
     
    if(k==12) //If random number is 12, print card with Q (Queen) on face
    {
    //Diamond Card
    printf("-------\n");
    printf("|%c    |\n", diamond);
    printf("|  Q  |\n");
    printf("|    %c|\n", diamond);
    printf("-------\n");
    k=10; //Set card value to 10
    }
     
    if(k==13) //If random number is 13, print card with K (King) on face
    {
    //Diamond Card
    printf("-------\n");
    printf("|%c    |\n", diamond);
    printf("|  K  |\n");
    printf("|    %c|\n", diamond);
    printf("-------\n");
    k=10; //Set card value to 10
    }
    return k;
}// End function
 
int heartcard() //Displays Heart Card Image
{
     
     
    srand((unsigned) time(NULL)); //Generates random seed for rand() function
    k=rand()%13+1;
     
    if(k<=9) //If random number is 9 or less, print card with that number
    {
    //Heart Card
    printf("-------\n");
    printf("|%c    |\n", heart); 
    printf("|  %d  |\n", k);
    printf("|    %c|\n", heart);
    printf("-------\n");
    }
     
    if(k==10) //If random number is 10, print card with J (Jack) on face
    {
    //Heart Card
    printf("-------\n");
    printf("|%c    |\n", heart);
    printf("|  J  |\n");
    printf("|    %c|\n", heart);
    printf("-------\n");
    }
     
    if(k==11) //If random number is 11, print card with A (Ace) on face
    {
    //Heart Card
    printf("-------\n");
    printf("|%c    |\n", heart);
    printf("|  A  |\n");
    printf("|    %c|\n", heart);
    printf("-------\n");
    if(player_total<=10) //If random number is Ace, change value to 11 or 1 depending on dealer total
         {
             k=11;
         }
          
         else
         {
             k=1;
         }
    }
     
    if(k==12) //If random number is 12, print card with Q (Queen) on face
    {
    //Heart Card
    printf("-------\n");
    printf("|%c    |\n", heart);
    printf("|  Q  |\n");
    printf("|    %c|\n", heart);
    printf("-------\n");
    k=10; //Set card value to 10
    }
     
    if(k==13) //If random number is 13, print card with K (King) on face
    {
    //Heart Card
    printf("-------\n");
    printf("|%c    |\n", heart);
    printf("|  K  |\n");
    printf("|    %c|\n", heart);
    printf("-------\n");
    k=10; //Set card value to 10
    }
    return k;
} // End Function
 
int spadecard() //Displays Spade Card Image
{
     
     
    srand((unsigned) time(NULL)); //Generates random seed for rand() function
    k=rand()%13+1;
     
    if(k<=9) //If random number is 9 or less, print card with that number
    {
    //Spade Card
    printf("-------\n");
    printf("|%c    |\n", spade);
    printf("|  %d  |\n", k);
    printf("|    %c|\n", spade);
    printf("-------\n");
    }
     
    if(k==10) //If random number is 10, print card with J (Jack) on face
    {
    //Spade Card
    printf("-------\n");
    printf("|%c    |\n", spade);
    printf("|  J  |\n");
    printf("|    %c|\n", spade);
    printf("-------\n");
    }
     
    if(k==11) //If random number is 11, print card with A (Ace) on face
    {
    //Spade Card
    printf("-------\n");
    printf("|%c    |\n", spade);
    printf("|  A  |\n");
    printf("|    %c|\n", spade);
    printf("-------\n");
    if(player_total<=10) //If random number is Ace, change value to 11 or 1 depending on dealer total
         {
             k=11;
         }
          
         else
         {
             k=1;
         }
    }
     
    if(k==12) //If random number is 12, print card with Q (Queen) on face
    {
    //Spade Card
    printf("-------\n");
    printf("|%c    |\n", spade);
    printf("|  Q  |\n");
    printf("|    %c|\n", spade);
    printf("-------\n");
    k=10; //Set card value to 10
    }
     
    if(k==13) //If random number is 13, print card with K (King) on face
    {
    //Spade Card
    printf("-------\n");
    printf("|%c    |\n", spade);
    printf("|  K  |\n");
    printf("|    %c|\n", spade);
    printf("-------\n");
    k=10; //Set card value to 10
    }
    return k;
} // End Function
 
int randcard() //Generates random card
{
      
                
     srand((unsigned) time(NULL)); //Generates random seed for rand() function
     random_card = rand()%4+1;
      
     if(random_card==1)
     {   
         clubcard();
         l=k;
     }
      
     if(random_card==2)
     {
         diamondcard();
         l=k;
     }
      
     if(random_card==3)
     {
         heartcard();
         l=k;
     }
          
     if(random_card==4)
     {
         spadecard();
         l=k;
     }    
     return l;
} // End Function   
 
void play() //Plays game
{
      
     int p=0; // holds value of player_total
     int i=1; // counter for asking user to hold or stay (aka game turns)
     char choice3;
      
     cash = cash;
     cash_test();
     printf("\nCash: $%d\n",cash); //Prints amount of cash user has
     randcard(); //Generates random card
     player_total = p + l; //Computes player total
     p = player_total;
     printf("\nYour Total is %d\n", p); //Prints player total
     dealer(); //Computes and prints dealer total
     betting(); //Prompts user to enter bet amount
        
     while(i<=21) //While loop used to keep asking user to hit or stay at most twenty-one times
                  //  because there is a chance user can generate twenty-one consecutive 1's
     {
         if(p==21) //If user total is 21, win
         {
             printf("\nUnbelievable! You Win!\n");
             won = won+1;
             cash = cash+bet;
             printf("\nYou have %d Wins and %d Losses. Awesome!\n", won, loss);
             dealer_total=0;
             askover();
         }
      
         if(p>21) //If player total is over 21, loss
         {
             printf("\nWoah Buddy, You Went WAY over.\n");
             loss = loss+1;
             cash = cash - bet;
             printf("\nYou have %d Wins and %d Losses. Awesome!\n", won, loss);
             dealer_total=0;
             askover();
         }
      
         if(p<=21) //If player total is less than 21, ask to hit or stay
         {         
             printf("\n\nWould You Like to Hit or Stay?");
              
             scanf("%c", &choice3);
             while((choice3!='H') && (choice3!='h') && (choice3!='S') && (choice3!='s')) // If invalid choice entered
             {                                                                           
                 printf("\n");
                 printf("Please Enter H to Hit or S to Stay.\n");
                 scanf("%c",&choice3);
             }
 
 
             if((choice3=='H') || (choice3=='h')) // If Hit, continues
             { 
                 randcard();
                 player_total = p + l;
                 p = player_total;
                 printf("\nYour Total is %d\n", p);
                 dealer();
                  if(dealer_total==21) //Is dealer total is 21, loss
                  {
                      printf("\nDealer Has the Better Hand. You Lose.\n");
                      loss = loss+1;
                      cash = cash - bet;
                      printf("\nYou have %d Wins and %d Losses. Awesome!\n", won, loss);
                      dealer_total=0;
                      askover();
                  } 
      
                  if(dealer_total>21) //If dealer total is over 21, win
                  {                      
                      printf("\nDealer Has Went Over!. You Win!\n");
                      won = won+1;
                      cash = cash+bet;
                      printf("\nYou have %d Wins and %d Losses. Awesome!\n", won, loss);
                      dealer_total=0;
                      askover();
                  }
             }
             if((choice3=='S') || (choice3=='s')) // If Stay, does not continue
             {
                printf("\nYou Have Chosen to Stay at %d. Wise Decision!\n", player_total);
                stay();
             }
          }
             i++; //While player total and dealer total are less than 21, re-do while loop 
     } // End While Loop
} // End Function
 
void dealer() //Function to play for dealer AI
{
     int z;
      
     if(dealer_total<17)
     {
      srand((unsigned) time(NULL) + 1); //Generates random seed for rand() function
      z=rand()%13+1;
      if(z<=10) //If random number generated is 10 or less, keep that value
      {
         d=z;
          
      }
      
      if(z>11) //If random number generated is more than 11, change value to 10
      {
         d=10;
      }
      
      if(z==11) //If random number is 11(Ace), change value to 11 or 1 depending on dealer total
      {
         if(dealer_total<=10)
         {
             d=11;
         }
          
         else
         {
             d=1;
         }
      }
     dealer_total = dealer_total + d;
     }
           
     printf("\nThe Dealer Has a Total of %d", dealer_total); //Prints dealer total
      
} // End Function 
 
void stay() //Function for when user selects 'Stay'
{
     dealer(); //If stay selected, dealer continues going
     if(dealer_total>=17)
     {
      if(player_total>=dealer_total) //If player's total is more than dealer's total, win
      {
         printf("\nUnbelievable! You Win!\n");
         won = won+1;
         cash = cash+bet;
         printf("\nYou have %d Wins and %d Losses. Awesome!\n", won, loss);
         dealer_total=0;
         askover();
      }
      if(player_total<dealer_total) //If player's total is less than dealer's total, loss
      {
         printf("\nDealer Has the Better Hand. You Lose.\n");
         loss = loss+1;
         cash = cash - bet;
         printf("\nYou have %d Wins and %d Losses. Awesome!\n", won, loss);
         dealer_total=0;
         askover();
      }
      if(dealer_total>21) //If dealer's total is more than 21, win
      {
         printf("\nUnbelievable! You Win!\n");
         won = won+1;
         cash = cash+bet;
         printf("\nYou have %d Wins and %d Losses. Awesome!\n", won, loss);
         dealer_total=0;
         askover();
      }
     }
     else
     {
         stay();
     }
      
} // End Function
 
void cash_test() //Test for if user has cash remaining in purse
{
     if (cash <= 0) //Once user has zero remaining cash, game ends and prompts user to play again
     {
        printf("You Are Bankrupt. Game Over");
        cash = 500;
        askover();
     }
} // End Function
 
int betting() //Asks user amount to bet
{
 printf("\n\nEnter Bet: $");
 scanf("%d", &bet);
 
 if (bet > cash) //If player tries to bet more money than player has
 {
        printf("\nYou cannot bet more money than you have.");
        printf("\nEnter Bet: ");
        scanf("%d", &bet);
        return bet;
 }
 else return bet;
} // End Function
 
void askover() // Function for asking player if they want to play again
{
    char choice1;
         
     printf("\nWould You Like To Play Again?");
     printf("\nPlease Enter Y for Yes or N for No\n");
     scanf("\n%c",&choice1);
 
    while((choice1!='Y') && (choice1!='y') && (choice1!='N') && (choice1!='n')) // If invalid choice entered
    {                                                                           
        printf("\n");
        printf("Incorrect Choice. Please Enter Y for Yes or N for No.\n");
        scanf("%c",&choice1);
    }
 
 
    if((choice1 == 'Y') || (choice1 == 'y')) // If yes, continue.
    { 
            system("cls");
            play();
    }
  
    else if((choice1 == 'N') || (choice1 == 'n')) // If no, exit program
    {
        fileresults();
        printf("\nBYE!!!!\n\n");
        system("pause");
        exit(0);
    }
    return;
} // End function
 
void fileresults() //Prints results into Blackjack.txt file in program directory
{
    FILE *fpresults; //File pointer is fpresults
    fpresults = fopen(RESULTS, "w"); //Creates file and writes into it
    if(fpresults == NULL) // what to do if file missing from directory
    {
               printf("\nError: File Missing\n");
               system("pause");
               exit(1);
    }
    else
    {     
     fprintf(fpresults,"\n\t RESULTS");
     fprintf(fpresults,"\n\t---------\n");
     fprintf(fpresults,"\nYou Have Won %d Times\n", won);
     fprintf(fpresults,"\nYou Have Lost %d Times\n", loss);
     fprintf(fpresults,"\nKeep Playing and Set an All-Time Record!");
    } 
     fclose(fpresults);
     return;
} // End Function
```

단순한 블랙잭 프로그램으로, 우리가 이 프로그램에서 millionares, 즉 백만장자에게만 돈을 줄 거라고 말하고 있다. 물론 우리가 타짜도 아니고, 정상적인 방법으로 블랙잭으로 백만장자가 되는 것은 불가능할 뿐더러 시간도 오래 걸릴 것이고, 문제가 의도하는 것도 그런 방법은 아닐 것이다. 코드를 살펴보며 취약한 부분을 살펴보자.

```c
int betting() //Asks user amount to bet
{
 printf("\n\nEnter Bet: $");
 scanf("%d", &bet);
 
 if (bet > cash) //If player tries to bet more money than player has
 {
        printf("\nYou cannot bet more money than you have.");
        printf("\nEnter Bet: ");
        scanf("%d", &bet);
        return bet;
 }
 else return bet;
} // End Function
```

bet의 cash 크기를 한 번 비교한 뒤, 만약 값이 현재 내 돈보다 적으면 다시 한 번 입력을 받는데, 이 값을 검사를 안하고 그냥 배팅으로 입력받는다. 즉 현재 돈보다 많은 돈을 써넣어 오류를 일으킨 뒤, 다시 한 번 더 많은 양의 돈을 입력하면 프로그램은 그 값을 그대로 베팅에 사용한다.  
그렇게 그냥 이기는 방식으로 해도 되지만, 더 쉬운 방식이 있다.  

```c
if(player_total<dealer_total) //If player's total is less than dealer's total, loss
      {
         printf("\nDealer Has the Better Hand. You Lose.\n");
         loss = loss+1;
         cash = cash - bet;
         printf("\nYou have %d Wins and %d Losses. Awesome!\n", won, loss);
         dealer_total=0;
         askover();
      }
```

이건 우리가 졌을 때 코드이다. 여기서 보면 우리가 지면 우리의 베팅만큼 우리의 돈에서 빼는 방식으로 진행된다. 그런데 우리의 베팅이 음수라면?  
위에서 보여줬다시피 베팅 당시에는 오직 현재 돈보다 베팅 금액이 큰 경우만을 조사하고, 그마저도 다음 입력에는 무력화된다. 그런데 음수는 처음부터 그냥 입력할 수 있는 것이다.  

그렇다면 베팅으로 int 범위 내의 큰 음수를 쓰고, 그대로 Stay해 져버리면? 그대로 그 돈이 굴러들어오는 것이다. 실행해보자.  

<img width="248" alt="image" src="https://user-images.githubusercontent.com/73513005/195349716-7e329915-f1e6-45db-9c1f-55470ebb5e51.png">

![image](https://user-images.githubusercontent.com/73513005/195349563-81da56ec-8914-49ea-ae3c-efb75d71d72c.png)

이렇게 지면서 승리하고, 돈과 flag를 동시에 챙길 수 있다.

