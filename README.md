<h1 align="center">Design of a Highly Configurable Cordic IP Core</h1>



<h3 align="left">Introduction:</h3>
<p align="left"> 
This project mainly focuses on the implementation of CORDIC Algorithm using Physical Design Flow. The word CORDIC is the abbreviation of Coordinate Rotation Digital Computer, which is an algorithm that is commonly used to calculate various mathematical functions, including trigonometric, logarithmic, and exponential functions. A CORDIC IP Core is a specialized hardware component that is designed to implement the CORDIC algorithm. The method can also be easily extended to compute square roots as well as hyperbolic functions.</p>
<p align="left">
The algorithm works by reducing the calculation into a number of micro-rotations for which the arctangent value is precomputed and loaded in a table. This method reduces the computation to addition, subtraction, compares, and shifts. All functions easily performed by FPGAs.
CORDIC cores are commonly used in digital signal processing (DSP) applications, such as in the design of filters, modulators, and demodulators. They can also be used in other applications that require high-speed and low-power processing of complex mathematical functions, such as in the fields of image and video processing, telecommunications, and cryptography.
</p>

<h3 align="left">CORDIC Theory:</h3>

- Let (x<sub>i</sub>, y<sub>i</sub>) and (x<sub>j</sub>, y<sub>j</sub>) are the coordinates and to rotate these coordinates I have performed the following matrix calculations. 

<img width="308" alt="image" src="https://user-images.githubusercontent.com/99958597/235726611-e9b34d2d-3c18-40cc-9ba1-3b61b129c968.png">

- The above equation can be modified by factoring a cosθ, then we will get the following, 
<img width="375" alt="image" src="https://user-images.githubusercontent.com/99958597/235728271-002f313f-44cc-4ccd-a40f-650093c7baab.png">

- So from the above calculation, we can say that any angle θ in the first quadrant can be represented with the help of the following equation. 

<img width="375" alt="image" src="https://user-images.githubusercontent.com/99958597/235731062-e7c69f1b-4497-40ab-a178-be49f4f2d5a1.png">

Where, S<sub>i</sub> = 0 or 1

- For instance,  the angle 0 is computed by letting S_i in equation (3) be simply S<sub>i</sub>=0,0,0,0... . The angle θ = pi/2 is computed by letting S<sub>i</sub> = 1,1,1,1. .. . pi/4 would be 1,0,0,0... , and so on.

- In practice, due to the accuracy limits of the digital representation, the summation can be limited to the number of bits representing the value, or fewer. The arctan (1/2<sup>n</sup>) quickly approaches to 0.

- **Note** that equation 3 can also be constructed such that S_i = -1 or 1. For example, the angle pi/4 would be computed with the sequence 1,1,−1,−1,−1,... . The advantage of this method will become apparent.

- This results in the following equation for tan θ<sub>n</sub>
<img width="375" alt="image" src="https://user-images.githubusercontent.com/99958597/235733187-4865c2e0-0dd4-4569-9cea-dd258a9b750a.png">

- Lets go back to the equation 2, , and given that the angle θ can be represented as a weighted sum (3). We can represent the rotation from (x<sub>i</sub> to  y<sub>i</sub>) as a sequence of smaller rotations where the n+1 rotational coordinate is derived from the n<sup>th</sup> coordinate using the following equation: 

<img width="410" alt="image" src="https://user-images.githubusercontent.com/99958597/235734096-7f3bf2c0-96f6-47b2-9c34-4b96e79851df.png">

- By replacing the equation 4 into equation 5 we are replacing the tangent function divided by 2<sup>n</sup>, which is a simple shift operatopn. 
<img width="422" alt="image" src="https://user-images.githubusercontent.com/99958597/235734960-d0534396-daa3-4cf8-a8c8-4c0473d256f1.png">

- By implementing the algorithm by letting S<sub>n</sub> with the values -1 and 1, and by doing that the cos θ will become, 
          
          cos (θ) = cos (-θ)      (7)
          
- The cos (θ) term is reduced to a constant K, which is given by the following equation, 
<img width="399" alt="image" src="https://user-images.githubusercontent.com/99958597/235736229-8f11d695-b3a4-4a9b-b8e9-2fe235e60e4a.png">

**As a result, the constant K indicates that the CORDIC function results in an overall gain of the factor 1/K, also known as the CORDIC gain. This may be considered in other areas of the system because it is a constant.**

<h3 align="left">IP CORE Description:</h3>

- A first quadrant coordinate rotational digital computer technique for computing transcendental functions are realised by the highly customizable CORDIC IP Core. The core can be implemented in RTL in one of three designs using definitions in the source:
<br>
<b>1. Combinatorial: </b>

- The combinatorial realisation sacrifices several levels of logic in order to solve equations in one clock cycle.
-  This option gives a result in a single clock cycle at the expense of very deep logic levels.
-  The combinatorial implementation runs at about 10 mhz while the iterative ones run at about 125 in a Lattice ECP2 device.
<br>
<b>2. Iterative: </b> 

- The problem is divided into a number of iterations using the iterative technique. Using this approach results in less logic, but it requires more clock cycles to finish. This option builds a single ROTATOR. 
- The user provides the arguments and gives the core ITERATIONS clock cycles to get the result.  A signal named init is instantiated to load the input values.  It uses the least amount of LUTs
<br>
<b>3. Pipelined: </b>

- Finally, the pipeline realization takes several clock cycles to complete, but the core can produce a new result on every clock cycle.
-  This option can take a new input on every clock and gives results ITERATIONS clock cycles later. It uses the most amount of LUTS.
<br>

<h3 align="left">Modes of Operation:</h3>

This CORDIC IP CORE operates in one of the two modes: 
<br>

<b>1. Rotate: </b>

- The user provides the core an angle in the rotate mode, and the core computes the sin and cos. Sin/Cos inputs are used. 
- The core enables computing angles in either radian or degree format in addition to core architecture. Additionally, there are provisions for dealing with variable bit lengths and iterations. The source has a thorough explanation of each of the many "defines."
-  In this mode the user supplies a X and Y cartesian vector and an angle. The CORDIC rotator seeks to reduce the angle to zero by rotating the vector.
-  To compute the cos and sin of the angle, set the inputs as follows:<br>

             y_in = 0;
             x_in = `CORDIC_1
             theta_in = the input angle 
             
             on completion: 

             y_out = sin
             x_out = cos
             
Where, The `CORDIC_1 above is the inverse of the cordic gain. 

<br>
<b>2. Vector: </b>

- In vector mode, and the computer outputs the arctangent. 
- In this mode the user supplies the tangent value in x and y and the rotator seeks to minimize the y value, thus computing the angle.
- To compute the arctan set the inputs as follows:
-  y_in and x_in  such that y/x = the tangent value for which you wish to find the angle 

             theta_in = 0;
             
             on completion
             
             theta_out = the angle
             
<h3 align="left">CORDIC IP CORE Implementation:</h3>
<h4 align="left">1. Functional Verification: </h4>
- This is the first step of the implementation of the CORDIC IP CORE, where I have written a piece of verilog code and performed functional verification by using a tool called <b>Cadence Incisive.</b>


![Screenshot from 2023-05-01 07-02-37](https://user-images.githubusercontent.com/99958597/235753209-039d3614-d0bb-4be5-9d07-2ddbc75dd4a0.png)

![Screenshot from 2023-05-01 06-54-53](https://user-images.githubusercontent.com/99958597/235754071-4ec93dfc-b2f2-4374-be8b-2f674e69ebbd.png)

- After compiling and elaborating the verilog design when it is free from all the errors, then we have verified the functionality of the design with the help of the waveforms generated. 

<h4 align="left">2. Gate Level Synthesis and Geneartion of Netlist: </h4>

- After successful verification of the functionality of the desired design then we need to generate the gate level netlist for the proceeding into the further steps. This step is performed by using a tool named <b>Cadence Genus.</b>  

![Screenshot from 2023-05-01 07-31-30](https://user-images.githubusercontent.com/99958597/235755081-d8870aea-407c-4826-9b0c-0b6554505e22.png)

![Screenshot from 2023-05-01 07-31-56](https://user-images.githubusercontent.com/99958597/235755572-b051724c-52e0-4ce0-b808-1ca3061dfa72.png)

![Screenshot from 2023-05-01 07-32-01](https://user-images.githubusercontent.com/99958597/235755646-6acf39ba-5441-41f3-a557-7ff27f8634b2.png)


- The above images represents the schematic of the CORDIC IP CORE. 
- To perfrom complete the synthesis process mainly we need three files. <br>

<b>1. Verilog Design File.</b><br>

<b>2. Input SDC File, describing all the design constraints and timing assignments.</b><br>

<b>3. A Library File, which contains all the standard cells information. This file will be provided by the Cadence tool itself.</b><br>

<b>4. A TCL Script to automate the process. This file is optional</b><br>

<h5 align="left"><b>Output  Files: </b></h5>

- The outputs files obtained in this Synthesis process will be, <br>

<b>1. Output SDC File.</b><br>

<b>2. A Netlist File.</b><br>

<b>3. Analysis Reports.</b><br>

<h4 align="left">3. Physical Design: </h4>

- This is the main step of this project, where we have performed the designing process by using the concepts like Floor Planning, Power Planning, Placement, Routing, and so on. To perform this step we have used a tool called <b>Cadence Innovus.</b>

<h5 align="left"><b>I. Floor Planning: </b></h5>

- Floorplanning is a crucial step in the physical design process of VLSI circuits. It involves arranging and placing the functional blocks of a digital design on a chip, while considering various design constraints such as area, power, timing, signal integrity, and manufacturability.
- The success of the overall physical design of the VLSI circuit depends heavily on the quality of the floorplan. 

![Screenshot from 2023-05-01 08-10-15](https://user-images.githubusercontent.com/99958597/235760502-7d0a4fef-257d-45e8-8129-8ecf1ba945a7.png)

![Screenshot from 2023-05-03 09-59-15](https://user-images.githubusercontent.com/99958597/235967187-fe56b0fe-c74e-4199-9faf-b03e53b9453b.png)



<h5 align="left"><b>II. Power Planning: </b></h5>

- Power planning involves dividing the chip into power domains, which are isolated from each other to avoid power distribution issues such as voltage drop, ground bounce, and noise coupling. Power Planning involves 3 steps:<br>

<b>1. Adding Power Rings.</b><br>

<b>2. Adding Power Strips.</b><br>

<b>3. Adding Special Routes.</b><br>

![Screenshot from 2023-05-01 11-54-58](https://user-images.githubusercontent.com/99958597/235967738-ee2b5bde-e200-4e91-b156-1639f61a85ba.png)

![Screenshot from 2023-05-03 10-04-55](https://user-images.githubusercontent.com/99958597/235967834-59f89e49-119b-4585-814d-942f3e2781ab.png)


<h5 align="left"><b>III. Placement: </b></h5>

- VLSI circuits that involves determining the optimal location of each functional block on the chip. The goal of placement is to minimize the total wire length between the functional blocks, while meeting the design constraints such as area, power, timing, and signal integrity.
- During placement, the functional blocks are initially placed randomly within the chip area. 

![Screenshot from 2023-05-01 08-25-34](https://user-images.githubusercontent.com/99958597/235762182-25c55b7e-0358-4bf4-b00b-1806f4dd4e6a.png)

![Screenshot from 2023-05-02 17-56-08 (copy)](https://user-images.githubusercontent.com/99958597/235762375-c94c251a-3366-4a45-875f-51edd9666653.png)

![Screenshot from 2023-05-03 10-13-31](https://user-images.githubusercontent.com/99958597/235967354-0797188d-2ee5-4798-bbf0-f3b9d256dda9.png)


<h5 align="left"><b>IV. Routing: </b></h5>

- Routing is a critical step in the physical design process of VLSI circuits that involves the creation of interconnects between the functional blocks placed on the chip. The goal of routing is to connect the input and output pins of the functional blocks in an optimal way, while satisfying various design constraints such as area, power, timing, and signal integrity.

<h5 align="left"><b>V. Static Timing Analysis: </b></h5>

- This step of is performed by using a tool called <b>Cadence Tempus.</b>
- STA is used to verify the timing constraints of the design, such as setup time, hold time, and clock period, and ensure that the design meets the desired performance specifications.

<img width="561" alt="image" src="https://user-images.githubusercontent.com/99958597/235765467-8a0038bb-9888-48cc-813f-11d00f47a5d6.png">

- Before Optimization: 
<img width="353" alt="image" src="https://user-images.githubusercontent.com/99958597/235764272-fb37ef79-6ce5-4dbb-82b0-efb25e1bb474.png">

- After Optimization: 
<img width="405" alt="image" src="https://user-images.githubusercontent.com/99958597/235764578-3237e891-04a5-494a-9c5f-6e8a59002c77.png">

<h3 align="left">Area Analysis:</h3>

<img width="537" alt="image" src="https://user-images.githubusercontent.com/99958597/235764947-6ec9a285-c383-48bc-98c9-a53c995b8743.png">

<img width="302" alt="image" src="https://user-images.githubusercontent.com/99958597/235765235-086cae51-5e90-4c16-b0ed-1f355ac50665.png">
          
<h3 align="left">Power Analysis:</h3>

<img width="548" alt="image" src="https://user-images.githubusercontent.com/99958597/235765633-291f0f8f-5101-4e49-a6ff-81d9cbefd64b.png">

<img width="607" alt="image" src="https://user-images.githubusercontent.com/99958597/235765749-ca7fd54c-c863-4350-a2ff-3239b9deb466.png">

<h3 align="left">Final Results:</h3>

![Screenshot from 2023-05-02 17-54-04](https://user-images.githubusercontent.com/99958597/235766189-195aba5b-27fe-476a-9f46-12657933af33.png)

![Screenshot from 2023-05-02 17-53-48](https://user-images.githubusercontent.com/99958597/235766256-514505a4-c58e-4ffc-bfad-7fbe3882a27e.png)

![Screenshot from 2023-05-01 10-09-23](https://user-images.githubusercontent.com/99958597/235766334-e53395b3-3fd1-497c-a39d-d889a6bd4e32.png)

<h5 align="left"><b>3D View of the Design: </b></h4>

![Screenshot from 2023-05-01 10-20-18](https://user-images.githubusercontent.com/99958597/235766623-a9122c48-a413-4932-adf6-d82c75ac30e4.png)

![Screenshot from 2023-05-01 10-20-43](https://user-images.githubusercontent.com/99958597/235766680-dbc75d31-4554-4c85-aeed-79c5a53acd7b.png)

<h3 align="left">Conclusion:</h3>

- Therefore, A highly configurable Cordic IP Core has been designed and various parameters has been analysed and observed. 
- The maximum frequency obtained in different architectures are listed below, 

<table align = "center">
        <thead>
            <tr>
                <th>Architecture</th><th>Maximum Frequency</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Combinational</td><td>9 MHz</td>
            </tr>
            <tr>
                <td>Iterative</td><td>125 MHz</td>
            </tr>
            <tr>
                <td>Pipelined</td><td>130 MHz</td>
            </tr>
        </tbody>
    </table>

<h5 align="left"><b>Design Summary: </b></h5>

<img width="202" alt="image" src="https://user-images.githubusercontent.com/99958597/235768906-8b8c0e6b-3d86-44bd-9752-eec933c3a5ad.png">

<h5 align="left"><b>Design Statistics:</b></h5>

<img width="295" alt="image" src="https://user-images.githubusercontent.com/99958597/235769155-ea8be8f9-4907-4a22-9d3f-ac1d536af276.png">

<h5 align="left"><b>I/O Port Summary:</b></h5>

<img width="464" alt="image" src="https://user-images.githubusercontent.com/99958597/235769385-e99b647c-6477-43a6-b98e-8f8128c9b88c.png">

<h3 align="left">EDA Tools Used:</h3>

- Xilinx Vivado
- Cadence Incisive 
- Cadence Genus
- Cadence Innovus
- Cadence Tempus 

<h3 align="left">Acknowledgement:</h3>

- KL University. 
- Dale Drinkard
- [Xilinx LogiCORE.](https://www.xilinx.com/products/intellectual-property/cordic.html#overview)


- <b>NOTE: We are not attatching any kind of design, testbench, tcl script, sdc files as we are suppossed to maintain the privacy of the project. If anyone have any doubts regarding the above design please feel free to drop a mail at santosh.achary0706@gmail.com or namgirisnehith13@gmail.com</b>
 



















