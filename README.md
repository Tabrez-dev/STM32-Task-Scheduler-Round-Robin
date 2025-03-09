# STM32 Task Scheduler - Round Robin

A custom-built round-robin task scheduler for the STM32F072RBT6, implemented completely from scratch by referencing the Cortex‑M0 user guide. This project is a testament to low-level embedded systems expertise, demonstrating manual context switching and real-time task management using the SysTick handler.

---

## Overview

This project implements a robust round-robin scheduler that manages multiple tasks by leveraging the Cortex‑M0’s SysTick timer. Every aspect—from initializing task stacks to context switching—is crafted manually, showcasing a deep understanding of ARM Cortex‑M0 architecture. Debugged meticulously using STM32CubeIDE, the scheduler offers a lightweight, efficient approach to task management in resource-constrained embedded systems.

---

## Technical Details

**SysTick Handler Usage:**  
At the core of this scheduler is the SysTick handler, which triggers every 1 ms to perform context switching. The handler saves the context of the current task by manually storing registers (R4–R11) on the task's Process Stack Pointer (PSP). It then selects the next task in a cyclic order, restores its context, and updates the PSP accordingly. Due to Cortex‑M0’s limitations (e.g., absence of multi-register load/store for high registers), all context save and restore operations are implemented using inline assembly with precise control over the processor registers. This approach ensures that each task resumes execution exactly where it left off.

**Key Techniques Implemented:**

- **Manual Context Switching:**  
  Using inline assembly to save and restore critical registers (R4–R11) without relying on hardware-supported multi-register instructions. This method provides full control over task execution and stack management.

- **Custom Task Stack Initialization:**  
  Each task stack is initialized with a dummy frame containing essential values like xPSR, PC, and LR. This setup ensures that, upon context restoration, the task resumes in thread mode with the correct execution context.

- **Round Robin Scheduling:**  
  The scheduler cycles through tasks using a simple modulo operation, ensuring each task gets a fair share of CPU time. This deterministic switching mechanism is ideal for systems requiring predictable and balanced task execution.

- **Debug and Trace via Semihosting:**  
  Debug messages printed to the console provide real-time feedback on task execution and context switching, with placeholders available for screenshots of console output to demonstrate system behavior during operation.

---

## Usage

1. **Development Environment:**  
   Open the project in STM32CubeIDE. The project is fully configured for the STM32F072RBT6.

2. **Building the Project:**  
   Simply build the project using STM32CubeIDE’s build functionality. The makefile and project configuration are set up to compile the scheduler along with its context switching routines.

3. **Flashing and Debugging:**  
   Connect your STM32 programmer/debugger, then flash the firmware onto the microcontroller. The semihosting output can be viewed in the debug console, showing task switching in action.

4. **Console Output:**  

![image](https://github.com/user-attachments/assets/964c4424-69ae-461c-ae25-d04be23ab91e)
![image](https://github.com/user-attachments/assets/668cde6e-0e9e-4cf8-949a-ac5dd31a38f5)


   The console output demonstrates the cyclic execution of tasks, with clear, periodic messages from each task handler.

---

## References

- **Cortex‑M0 Technical Reference Manual**
- **STM32F072RBT6 Datasheet**
- **STM32CubeIDE Documentation**

---

*Created by [Tabrez-dev](https://github.com/Tabrez-dev) – March 2025*
