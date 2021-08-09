# stm8BusOff

STM8 as BUS OFF fast and slow recovery strategy

This article mainly introduces you to the STM8 for BUS OFF speed recovery strategy, the main content includes basic applications, practical skills, principles and mechanisms, etc., I hope it will be helpful to everyone.

An error on the CAN bus will cause the CAN controller to enter the BUS OFF state. For details, please refer to the CAN specification. code

CAN controller provides automatic recovery and manual recovery functions. it

1. Automatic recovery io

Auto-recovery is relatively simple, turn on the auto-recovery function during initialization. If the requirements are not high, it is recommended to turn it on, otherwise CAN BUS OFF will not be able to restore communication. event

/*atuo bus off recovery */
CAN_MasterCtrl=CAN_MasterCtrl_AutoBusOffManagement | CAN_MasterCtrl_NoAutoReTx;
2. Manually restore ast

Usually the car factory requires that the ECU cannot be restored automatically, but restores quickly and then slowly. 

It is often used: first 100ms to restore 5 times, and then 1000ms to restore once. 

Configuration

(1) occurs BUS OFF, immediately close the TX, and then reset the CAN controller summary

(2) Fast recovery times +1

(3) If the number of quick recovery times is less than 5, set the recovery time to 100ms, otherwise, set the recovery time to 1000ms

(4) When the recovery time is up, turn on TX

(5) The number of quick recovery times will be cleared if the message is successfully sent

Open related interrupts during initialization

  CAN_ITConfig(CAN_IT_FMP,ENABLE); 
  CAN_ITConfig(CAN_IT_BOF,ENABLE); 
  CAN_ITConfig(CAN_IT_ERR,ENABLE); 
Write like this in interrupt

INTERRUPT_HANDLER(CAN_TX_IRQHandler, 9)
{
  /* In order to detect unexpected events during development,
     it is recommended to set a breakpoint on the following instruction.
  */
  
  if(CAN_GetITStatus(CAN_IT_BOF) == SET)
  {
    CAN_CancelTransmit(CAN_TransmitMailBox_0);
    CAN_CancelTransmit(CAN_TransmitMailBox_1);
    CAN_CancelTransmit(CAN_TransmitMailBox_2);
    CAN_ClearITPendingBit(CAN_IT_BOF);
  }
  else if(CAN_GetITStatus(CAN_IT_TME) == SET)
  {
    CAN_ClearITPendingBit(CAN_IT_TME);
  }
  
}
Note that CAN_CancelTransmit must be used here, otherwise there will be problems

Summarize:

1. The relevant interrupt must be turned on

2. BUS OFF automatic manual recovery needs to be configured to

3. To cancel the current sending data when BUS OFF occurs
