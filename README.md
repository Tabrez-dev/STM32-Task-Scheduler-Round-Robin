# STM32 Task Scheduler - Round Robin

A custom-built round-robin task scheduler for the STM32F072RBT6, implemented completely from scratch by referencing the Cortex‑M0 user guide.Demonstrating manual context switching and real-time task management using the SysTick handler.

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




This project’s execution flow and design leverage a combination of standard C functions and naked functions (which omit the automatic prologue/epilogue) to achieve low-level context switching. The numbering below follows the order in which functions are executed and interact:

1. **main()**  
   *Type: Standard C function*  
   **Role:**  
   - Entry point of the program.
   - Initializes debugging support via `initialise_monitor_handles()`.
   - Sets up the scheduler’s main stack by calling **init_scheduler_stack()**.
   - Captures task handler addresses for each task.
   - Initializes each task’s stack using **init_tasks_stack()**.
   - Configures the SysTick timer with **init_systick_timer()** for a 1ms tick.
   - Switches the CPU’s active stack pointer from MSP to PSP using **switch_sp_to_psp()**.
   - Launches the first task by calling one of the task handlers (e.g., **task1_handler()**).

2. **initialise_monitor_handles()**  
   *Type: External function (provided by the debug environment)*  
   **Role:**  
   - Sets up semihosting, allowing console output for debugging purposes.

3. **init_scheduler_stack(uint32_t sched_top_of_stack)**  
   *Type: Naked function*  
   **Role:**  
   - Directly initializes the Main Stack Pointer (MSP) to a predefined scheduler stack location without compiler-added register saving.

4. **init_tasks_stack(void)**  
   *Type: Standard C function*  
   **Role:**  
   - Iterates over all tasks to initialize their individual stacks.
   - Creates a dummy stack frame for each task that includes:
     - The initial xPSR value.
     - The task’s PC (set to the task handler’s address).
     - The LR value (set to an EXC_RETURN constant for thread mode using PSP).
   - Ensures proper stack alignment for reliable execution.

5. **init_systick_timer(uint32_t tick_hz)**  
   *Type: Standard C function*  
   **Role:**  
   - Configures the SysTick timer registers (RVR, CSR, and CVR) to generate periodic interrupts at a rate defined by `tick_hz` (1ms in this case).
   - Enables the SysTick interrupt and selects the processor clock as the source.

6. **switch_sp_to_psp(void)**  
   *Type: Naked function*  
   **Role:**  
   - Switches the active stack pointer from MSP (used during initialization) to PSP (used by tasks).
   - Retrieves the current task’s PSP value, sets the PSP register, and updates the CONTROL register accordingly—all while preserving LR manually.

7. **Task Handlers (task1_handler, task2_handler, task3_handler, task4_handler)**  
   *Type: Standard C functions*  
   **Role:**  
   - Represent individual tasks that execute continuously (e.g., printing a message).
   - Provide a simple workload to demonstrate that context switching occurs as intended.

8. **SysTick_Handler(void)**  
   *Type: Naked function*  
   **Role:**  
   - Serves as the interrupt service routine for the SysTick timer.
   - **Context Saving:**  
     - Reads the current PSP.
     - Manually decrements the PSP and stores registers R4 through R11 (using inline assembly) to save the context of the running task.
   - **Task Switching:**  
     - Calls helper functions to save the current task’s PSP, update the task index (round-robin), and retrieve the next task’s PSP.
   - **Context Restoring:**  
     - Loads registers R4–R11 from the new task’s stack.
     - Adjusts the PSP back and updates the PSP register.
   - Returns control to the new task by restoring the saved LR.

9. **save_psp_value(uint32_t current_psp_value)**  
   *Type: Standard C function*  
   **Role:**  
   - Saves the current task’s Process Stack Pointer (PSP) into an array that maintains the context for all tasks.

10. **update_next_task(void)**  
    *Type: Standard C function*  
    **Role:**  
    - Updates the index of the current task using a modulo operation, ensuring a cyclic (round-robin) scheduling mechanism.

11. **get_psp_value(void)**  
    *Type: Standard C function*  
    **Role:**  
    - Retrieves the stored PSP value for the currently active task from the task context array.




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

