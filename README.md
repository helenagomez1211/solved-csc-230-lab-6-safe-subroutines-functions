Download Link: https://assignmentchef.com/product/solved-csc-230-lab-6-safe-subroutines-functions
<br>
Recall from the previous lab, a function call implies a transfer (branch, jump) to an address representing the entry point of the function, causing execution of the body of the function. When the function is finished, another transfer occurs to resume execution at the statement following the original call. The first transfer is the function call (for invocation); the second transfer is the return. Together, this constitutes the processor’s call-return mechanism for subroutine execution.

During the subroutine execution, the corresponding code would likely need to use some registers. We would NOT want that code to override the information stored in the registers by the caller.

How to determine which registers are <em>safe</em> to use?

The best practice is to protect the caller’s registers by saving the information stored there on the stack. After the subroutine completes execution, that information can be easily restored. In other words, we can back-up the registers using the stack before executing the subroutine and restore them afterwards.

Who protects the registers, the caller or the subroutine?

Arguably, the most convenient option is for the subroutine to protect any registers that it uses.

Following is an example of a <em>safe</em> subroutine which does not produce any undesirable <em>side effects</em>. This subroutine copies a c-string from the program memory location labeled “msg” to the data memory location labeled “msg_copy”.

<strong>Example</strong>: Create an Assembler project in Atmel Studio 7 and replace the contents of its main.asm with the program provided in copy_string.asm file on conneX. Build the program and debug it. Observe that the initial values in r16 and r30 are changed while the subroutine executes, but then they are restored before the program reaches the end-loop (“done: rjmp done”). Verify the contents of the stack as the program executes and the values are pushed onto it.

Below is the diagram of the internal data memory (SRAM) when “lpm r16, Z+” is executed for the first time.




<table width="0">

 <tbody>

  <tr>

   <td width="79">Address</td>

   <td width="84">content</td>

   <td width="146">details</td>

   <td width="333">notes</td>

  </tr>

  <tr>

   <td width="79">0x0000 ~0x001F</td>

   <td width="84"> </td>

   <td width="146">General Purpose Registers</td>

   <td width="333"> </td>

  </tr>

  <tr>

   <td width="79">0x0020 ~0x005F</td>

   <td width="84"> </td>

   <td width="146">64 I/O Registers</td>

   <td width="333"> </td>

  </tr>

  <tr>

   <td width="79">0x0060 ~0x01FF</td>

   <td width="84"> </td>

   <td width="146">416 extended I/ORegisters</td>

   <td width="333"> </td>

  </tr>

  <tr>

   <td width="79">0x0200</td>

   <td width="84"> </td>

   <td width="146"> </td>

   <td width="333">.DSEG</td>

  </tr>

  <tr>

   <td width="79"> … </td>

   <td width="84"> </td>

   <td width="146"> </td>

   <td width="333"> </td>

  </tr>

  <tr>

   <td width="79">0x21F7</td>

   <td width="84"> </td>

   <td width="146"> &lt;-SP</td>

   <td width="333">“0x21F7” is stored in the stack pointer register</td>

  </tr>

  <tr>

   <td width="79">0x21F8</td>

   <td width="84">R16</td>

   <td width="146">  saved register</td>

   <td rowspan="5" width="333">In the subroutine “copy_string”, the registers to be used by the subroutine are pushed onto the stack to protect them from being changed.</td>

  </tr>

  <tr>

   <td width="79">0x21F9</td>

   <td width="84">XL (R26)</td>

   <td width="146">  saved register</td>

  </tr>

  <tr>

   <td width="79">0x21FA</td>

   <td width="84">XH (R27)</td>

   <td width="146">  saved register</td>

  </tr>

  <tr>

   <td width="79">0x21FB</td>

   <td width="84">ZL (R30)</td>

   <td width="146">  saved register</td>

  </tr>

  <tr>

   <td width="79">0x21FC</td>

   <td width="84">ZH (R31)</td>

   <td width="146">  saved register</td>

  </tr>

  <tr>

   <td width="79">0x21FD</td>

   <td width="84">high(ret)</td>

   <td width="146">  return address</td>

   <td rowspan="3" width="333">The CPU pushes the return address onto the stack automatically when “call copy_string” is executed.</td>

  </tr>

  <tr>

   <td width="79">0x21FE</td>

   <td width="84">mid(ret)</td>

   <td width="146">  return address</td>

  </tr>

  <tr>

   <td width="79">0x21FF</td>

   <td width="84">low(ret)</td>

   <td width="146">  return address</td>

  </tr>

 </tbody>

</table>




Recall that ATmega2060 has 256KB of word-addressable program memory, and thus the program counter (PC) needs to count as high as 128K word addresses (2<sup>17</sup> words) (refer to page 7, Table 2-1 of the datasheet). Therefore, the PC register uses 3 bytes, which are stored on the stack when a call instruction executes. Then, the called subroutine stores the protected registers on top of the return address. After the registers are restored, the return address is on the top of the stack again when the ret instruction executes and writes the top three bytes on the stack to the PC register, thereby returning the control flow back to the instruction which immediately follows the initial call instruction.

<ol>

 <li><strong> Passing (and returning) parameters via the stack.</strong></li>

</ol>

There are two ways to pass parameters to a subroutine: <em>by value</em> and <em>by reference</em>. Below are two examples illustrating each method.

<strong>Example 1 (</strong><strong><em>pass by value</em>):</strong>

Consider the following C function that takes two 8-bit unsigned integers as parameters, multiplies them, and returns the result.

uint16_t multiply(uint8_t multiplicand, uint8_t multiplier) {

uint16 result = multiplicand * multiplier;

return result;

}

An equivalent Assembly function is provided in the multiply_by_value.asm file on conneX. Here, the caller (main program) pushes the 8-bit values of the two parameters onto the stack and also reserves the required 16-bit space on the stack for the return value. The function protects all registers that it uses, retrieves the parameters from the stack, multiplies the integers, and stores the result in the reserved location on the stack before returning.

Download the multiply_by_value.asm file and put it in the same location as main.asm for the project that you started earlier in this lab. Then, add the file to your project by right-clicking the lab in the Solution Explorer window and choosing “Add-&gt;Existing item…”:

Then, right-click the file and choose “Set As EntryFile”:

Only one file can be set as EntryFile at any time; this indicates which Assembly file will be built and debugged by the Atmel Studio.

Build and debug the solution. Observe the stack memory and SP as you step through the program. For convenience, use a breakpoint at the instruction immediately after the multiply loop and the debugger’s “run” command to skip over the loop execution. Below is the stack frame diagram of the stack contents when the instruction “in ZH, SPH” is executed. An equivalent diagram is also available in the comments right before the function code in the multiply_by_value.asm file.




<table width="0">

 <tbody>

  <tr>

   <td width="66">Address</td>

   <td width="104">content</td>

   <td width="139">details</td>

   <td width="333">notes</td>

  </tr>

  <tr>

   <td width="66">0x0200</td>

   <td width="104"> </td>

   <td width="139"> </td>

   <td width="333">.DSEG</td>

  </tr>

  <tr>

   <td width="66"> … </td>

   <td width="104"> </td>

   <td width="139"> </td>

   <td width="333"> </td>

  </tr>

  <tr>

   <td width="66">0x21F1</td>

   <td width="104"> </td>

   <td width="139"> &lt;-SP</td>

   <td width="333">“0x21F1” is stored in the stack pointer register</td>

  </tr>

  <tr>

   <td width="66">0x21F2</td>

   <td width="104">zero</td>

   <td width="139">saved register R0</td>

   <td rowspan="7" width="333">The registers to be used by the “multiply” subroutine are pushed onto the stack at the very beginning of the subroutine to protect their values from being changed by the subroutine.</td>

  </tr>

  <tr>

   <td width="66">0x21F3</td>

   <td width="104">result_low</td>

   <td width="139">saved register R4</td>

  </tr>

  <tr>

   <td width="66">0x21F4</td>

   <td width="104">result_high</td>

   <td width="139">saved register R3</td>

  </tr>

  <tr>

   <td width="66">0x21F5</td>

   <td width="104">multiplier</td>

   <td width="139">saved register R2</td>

  </tr>

  <tr>

   <td width="66">0x21F6</td>

   <td width="104">multiplicand</td>

   <td width="139">saved register R1</td>

  </tr>

  <tr>

   <td width="66">0x21F7</td>

   <td width="104">ZH</td>

   <td width="139">saved register R30</td>

  </tr>

  <tr>

   <td width="66">0x21F8</td>

   <td width="104">ZL</td>

   <td width="139">saved register R31</td>

  </tr>

  <tr>

   <td width="66">0x21F9</td>

   <td width="104">high(ret)</td>

   <td width="139">return address</td>

   <td rowspan="3" width="333">The CPU pushes the return address (the address of the next command) onto the stack automatically when “call add_num” is executed.</td>

  </tr>

  <tr>

   <td width="66">0x21FA</td>

   <td width="104">mid(ret)</td>

   <td width="139">return address</td>

  </tr>

  <tr>

   <td width="66">0x21FB</td>

   <td width="104">low(ret)</td>

   <td width="139">return address</td>

  </tr>

  <tr>

   <td width="66">0x21FC</td>

   <td width="104">0x00</td>

   <td width="139">result_high</td>

   <td rowspan="2" width="333"> </td>

  </tr>

  <tr>

   <td width="66">0x21FD</td>

   <td width="104">0x00</td>

   <td width="139">result_low</td>

  </tr>

  <tr>

   <td width="66">0x21FE</td>

   <td width="104">0xCD</td>

   <td width="139">parameter 2</td>

   <td rowspan="2" width="333">The caller (in this case the main program) pushes these parameters onto the stack.</td>

  </tr>

  <tr>

   <td width="66">0x21FF</td>

   <td width="104">0xAB</td>

   <td width="139">parameter 1</td>

  </tr>

 </tbody>

</table>




<strong>Example 2 (</strong><strong><em>pass by reference</em>)</strong>:




Below is the stack frame diagram for a very similar multiplication function as above, except instead of the values, their corresponding memory addresses are passed to the multiply subroutine.

<strong> </strong>

<table width="0">

 <tbody>

  <tr>

   <td width="66">Address</td>

   <td width="104">content</td>

   <td width="139">details</td>

   <td width="333">notes</td>

  </tr>

  <tr>

   <td width="66">0x0200</td>

   <td width="104"> </td>

   <td width="139"> </td>

   <td width="333">.DSEG</td>

  </tr>

  <tr>

   <td width="66"> … </td>

   <td width="104"> </td>

   <td width="139"> </td>

   <td width="333"> </td>

  </tr>

  <tr>

   <td width="66"> </td>

   <td width="104"> </td>

   <td width="139"> </td>

   <td width="333"> </td>

  </tr>

  <tr>

   <td width="66"> </td>

   <td width="104"> </td>

   <td width="139"> </td>

   <td width="333"> </td>

  </tr>

  <tr>

   <td width="66"> </td>

   <td width="104"> </td>

   <td width="139"> &lt;-SP</td>

   <td width="333">“0x21F1” is stored in the stack pointer register</td>

  </tr>

  <tr>

   <td width="66">0x21EE</td>

   <td width="104">zero</td>

   <td width="139">saved register R0</td>

   <td rowspan="9" width="333">The registers to be used by the “multiply” subroutine are pushed onto the stack at the very beginning of the subroutine to protect their values from being changed by the subroutine.</td>

  </tr>

  <tr>

   <td width="66">0x21EF</td>

   <td width="104">result_low</td>

   <td width="139">saved register R4</td>

  </tr>

  <tr>

   <td width="66">0x21F0</td>

   <td width="104">result_high</td>

   <td width="139">saved register R3</td>

  </tr>

  <tr>

   <td width="66">0x21F1</td>

   <td width="104">multiplier</td>

   <td width="139">saved register R2</td>

  </tr>

  <tr>

   <td width="66">0x21F2</td>

   <td width="104">multiplicand</td>

   <td width="139">saved register R1</td>

  </tr>

  <tr>

   <td width="66">0x21F3</td>

   <td width="104">YH</td>

   <td width="139">saved register R28</td>

  </tr>

  <tr>

   <td width="66">0x21F4</td>

   <td width="104">YL</td>

   <td width="139">saved register R29</td>

  </tr>

  <tr>

   <td width="66">0x21F5</td>

   <td width="104">ZH</td>

   <td width="139">saved register R30</td>

  </tr>

  <tr>

   <td width="66">0x21F6</td>

   <td width="104">ZL</td>

   <td width="139">saved register R31</td>

  </tr>

  <tr>

   <td width="66">0x21F7</td>

   <td width="104">high(ret)</td>

   <td width="139">return address</td>

   <td rowspan="3" width="333">The CPU pushes the return address (the address of the next command) onto the stack automatically when “call add_num” is executed.</td>

  </tr>

  <tr>

   <td width="66">0x21F8</td>

   <td width="104">mid(ret)</td>

   <td width="139">return address</td>

  </tr>

  <tr>

   <td width="66">0x21F9</td>

   <td width="104">low(ret)</td>

   <td width="139">return address</td>

  </tr>

  <tr>

   <td width="66">0x21FA</td>

   <td width="104">0x02</td>

   <td width="139">address of answer</td>

   <td rowspan="6" width="333">The caller (in this case the main program) pushes these addresses onto the stack in big-endian format (most significant byte in the lower memory location than the least significant byte).</td>

  </tr>

  <tr>

   <td width="66">0x21FB</td>

   <td width="104">0x02</td>

   <td width="139">address of answer</td>

  </tr>

  <tr>

   <td width="66">0x21FC</td>

   <td width="104">0x02</td>

   <td width="139">address num2</td>

  </tr>

  <tr>

   <td width="66">0x21FD</td>

   <td width="104">0x01</td>

   <td width="139">address num2</td>

  </tr>

  <tr>

   <td width="66">0x21FE</td>

   <td width="104">0x02</td>

   <td width="139">address num1</td>

  </tr>

  <tr>

   <td width="66">0x21FF</td>

   <td width="104">0x00</td>

   <td width="139">address num1</td>

  </tr>

 </tbody>

</table>

<strong> </strong>

As in the previous example, download the multiply_by_reference.asm file, put it in the same location as main.asm and the multiply_by_value.asm files, add it to your Atmel Studio solution, and set it as “EntryFile”. Build and debug the solution. Observe the stack memory and SP as you step through the program. The stack frame diagram above illustrates the stack contents when the instruction “in ZH, SPH” is executed. An equivalent diagram is also available in the comments right before the function code in the multiply_by_value.asm file.

<strong>  </strong>

<ul>

 <li></li>

</ul>

<strong> </strong>

You may find it helpful to use the stack_frame.docx file provided on conneX or a piece of scrap paper for designing and tracing the stack when answering the questions below.

<ol>

 <li>For this exercise you will write a modified version of the copy_string function in the main.asm file (from the first section of this lab). First, design the stack for it and then modify this function to take the program memory address and the data memory address via the stack instead of using a pre-defined memory location as it does now. This way you can re-use this code in the future to copy any string(s) from program memory to data memory.</li>

 <li>Extend the program from the previous exercise. First, design the stack for a new function called string_length, and then write the function. This function should take a parameter via the stack (in big-endian format) which contains the data memory address where a c-string resides. The function should count the number of characters in that string and return it by via the stack. Remember, the caller has to reserve the space on the stack for the return value.</li>

 <li>Use the Word file provided on conneX or a blank piece of paper and trace the stack for the recursive function in the triangle_number.asm file, which is located on conneX, from the beginning until the program finishes by reaching the end-loop (“done: rjmp done”). What does the program do?</li>

</ol>

<strong> </strong>