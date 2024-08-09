# The Hypersonic-Inlets Repository is a collection of code made for Hyperstats William & Mary Summer Research.
### References and usage credit can be given to Ethan Gaul and Maddie Sablan

Results of testing turned into Air Force in conjuction with NASA Langley
### How to use:

Final 2 codes for submission are tabbed with 'Finished code' and both serve different purposes:
'Optimizer_for_any_#_of_angle_with_adaptive_neural_network'

'Optimizer_for_any_#_of_angle_with_neural_network_condensor'
#
To use inlet optimizing code with no computational resource contraints
  use the 'angle optimization with adaptive neural network' code
# 
To use inlet optimization with computational resource contraints (slight accuracy loss)
  use the 'angle optimization with neural network condensor' code
### Proof of Concept Code
'Automated Optimal Angle Finder for Vehicle Underbelly' : Initial test for a fully automated user interface with 4 independant variable

'User_Interface_ALPHACODE' : Final prototype for a fully automated user interface

'Surrogate_Model_ALPHACODE' : Final prototype for the neural network and SOLIDWORKS automation with a 4 angle input

'Single and Double Angle Adjustments to Mach Number and Pressure at Inlet' : Inital trend testing, 2 angle combinations

'Initial_Prototyping_and_Graphs' : Secondary testing to find trends in our data, includes many angle combinations

#
Coding Research:

Hypersonic inlets have many factors to account for when designing, which makes it challenging to do. An inherent issue of a hypersonic inlet is its relationship between air speed and pressure. Generally speaking, as airspeed decreases through the inlet, relative pressure also decreases. Since the combustor and injector are optimized by having a higher relative pressure and lower air speed, this paper discusses how we achieve these conditions (2.5 Mach number through the combustor with an input Mach number of 5; pressure ratio as close to 1 as possible to reflect minimal loss) by adjusting the inlet geometry and shape. We define our model as efficient when it is able to produce results where Mach speed is 2.50 (with significant figures) and pressure ratio is maximized.
Currently there are models and equations in place to assist with the process of inlet creation; however, these models are complicated and are time consuming to complete. Through the use of machine learning modules, we can create simplified approximation models of these intricate equations that speed up testing time and make the inlet design process user friendly.
In order to simplify our given complicated functions, we need to set up and train a neural network. To set up our network, we need some kind of data for the machine to learn from and gather trend information to make informed guesses. To do this, we created a ‘random’ sample code that produced useful training data. Random is in apostrophes because the level of randomness is controlled in order to give the machine range-constrained data that is close to the desired Mach number. It does this by passing random values (0.01 to an upper limit) on each array angle and then only appending arrays with sums 30 to 40 to the training matrix. This will ideally produce only arrays with Mach numbers close to 2.5 (as found in Figure 6 above).
The upper limit of each angle is important to include since we need to stay within the domain of the original functions we are training on. For example, since we cannot exceed 90 degrees, each angle in a 3-angle array should not exceed 30 degrees. It is important to note that this method would not work if the shape was constrained to 90 degrees; however, since the optimal sum is 36 as mentioned before, the error is negligible when accounting for the shape at those higher sums. The optimal angles in that scenario would look like [7, 13, 16]. Notice how each angle isn’t even close to the upper limit of 30 degrees since the target sum is 36. In other words, at higher angle amounts will result in the curve becoming more linear, but since the target is much smaller than the allotted amount of 90, the difference will account for this loss of shape. The equation for the upper limit was experimentally found by plugging in different numbers for different angle amounts until a math domain error was returned.
The sample size for the random data was set to 5000. We found that this value allowed for accurate results without sacrificing too much time for training. Now that our sampling is set up, it is time to define the model architecture. We used the Keras python package and built a sequential model neural networkl using their tools. Two different models were built to output Mach number and pressure with the same input. Since the number of parameters were subject to change (due to the number of angles and dimensionality changing), we used plenty of neurons on each layer to account for smaller models being trained. This way, there is no noticeable learning deficit when looking at the range of the model. We were still having issues with our model getting stuck (no loss change) when being trained, so batch normalization layers were added as well as a final dropout layer. This completely rids the model of any issues during training.
Now that the model is set up and has its training data, it is time for it to learn. We faced some issues when approaching the training task such as save data, optimal epoch counts, and user-friendly definitions. 
	Since the dimensionality of our neural network can change for every run that we do, there is no ‘one size fits all’ approach to this problem. Because of this we cannot include a save data callback to our fitting function. If we did, we would get an error when trying to apply our saved model to a scenario that has a different number of angles as the input. This problem makes it so that we have to redo model training for every test that we would like to do. Theoretically this can be worked around, however, which we will discuss later.
	The number of epochs, or training renditions, that we used for most of our code building was 300. This number was doubled later for better model accuracy when creating graphs and presentable data. This is because the 300 epochs were returning good answers only some of the time, and doubling the time of completion was deemed acceptable for an increase in reliability. We found the correct fit for the number of epochs needed for training experimentally. By plugging in different training epochs for the same angle numbers we can measure how accurate each epoch count really is. In Figure 10 below there is a visualization of the results of this experimentation. It is clear to see as epochs go below 600 in this case, the reliability of accuracy decreases. Additionally, as epochs increase past 600 there may be a slight decrease in accuracy, but it is a tradeoff with an increase in reliability. Because of this, 600 epochs were used for further experimentation.
We decided to use the NLopt (non-linear optimization) package since we have 2 different converging functions that we need to find the maximum for. The syntax for the NLopt optimize function has many components. I will cover the design decisions for the selected algorithm, bounds, and tolerances. 
The non-linear optimization function has many different algorithms to choose from, each with different behaviors and results. There are many to choose from, but there are several parameters to help us pick which algorithm is best suited for our optimization. These parameters include function nature, size, and input accuracy. Function nature is how the objective function behaves with changing variables and deals with the complexity of its output. Since our J theta function is a first-degree polynomial, we can rule that our function nature is quite simple; thus, allowing us to choose from NLopt algorithms that are less computationally advanced. The range of our function (that we need) is relatively small (Mach numbers between 2.3 and 2.8), so we can use an algorithm that does a narrow search which will save time for the user during optimization. Since we used a sophisticated and calibrated equation for our guessing function, the difference between the guessed solution and the real solution are very minimal. Because of this, we use an optimizer algorithm that relies on a small differential. All of these characteristics allow us to minimize computation resources and time which supports our goal of a user-friendly code. We used the LN_COBYLA algorithm for our optimizer. The LN stands for derivative-free local optimization while COBYLA stands for constrained optimization by linear approximation. This combination of algorithms allows for a speedy discovery of function intersections. Other options such as GD, global gradient-based optimization, are more resource intensive and are constrained by dimension. We can save computer resources by having our initial guess near the global maximum and telling the computer to look for a local maximum (which, in turn, is the global maximum), rather than having a bad guess and telling the computer to look for a global maximum.
The COBYLA part of our optimizing algorithm requires a constraint, or bound, for the input. In this case, the bounds of our angle input are the math domain of our original function. More specifically, the sum of our angles cannot be greater than 90 and each angle must be above 0. As mentioned before, the higher bound for each angle cannot be constant since the dimensionality of our input is changing frequently with each run. For example, in order to satisfy the constraints, a 3-angle array mustn’t have an angle above 30 degrees (to not exceed 90 degrees) while a 6-angle array mustn’t have an angle above 15 degrees. Once again, Equation 1 was used and the results of testing in Figure 5 provided useful information on angle constraints.
After all these syntax characteristics are satisfied, the optimizer function still will not converge in reasonable time. In order to improve our code further, we added tolerance functions. These allow the convergence to rely on close proximity rather than exact intersections. There will be a loss in accuracy based on the constant used in tolerance, so multiple tests were conducted to find the sweet spot of our final Mach number maintaining 3 matching significant figures to our desired number. The tolerance number we ended up getting was 1×10^(-7) meaning our J theta values of each neural network can get that close to each other to be considered equal. 
With these parameters in place, our optimizer plugs both models into the objective function and finds the convergence of both. This convergence location is where pressure is maximized, and Mach number is 2.50. It will plug the optimized angles into the Mach/pressure function and compare the answers to the known solution for that angle set. Lastly, it will print the accuracy for user assurance. We have gotten a consistent 99% at least for each run with most runs returning a 99.9% accuracy.
We found that as the number of angles increases, the pressure ratio also increases. A tradeoff of this relationship is that the accuracy gets more random as the number of angles increases. This is because the dimension of our input increases when we increase our angle number; thus, the model gains parameters and therefore becomes more complicated. This requires more computational allocation when dimensionality increases which results in less predictable accuracy.
Now that our code is functional and produces acceptable results, it is time to reduce computation time and resources needed. Different methods we used to make our code more efficient include sample size testing and angle splitting. 
By reducing the sample size we use in our random training matrix, we can decrease the time it takes for the models to learn from the data. Additionally, less resources are used to create the matrix; thus, deeming this design decision efficient. As we decrease sample points, however, the model has less points to learn from and can become less accurate. We can find the optimal point where accuracy is acceptable, and size is decreased for our solution by plotting our accuracy tests. This optimized point was found at 3000 sample arrays. Although it looks like 1000 points are the best in the graphs, accuracy decreases after that point (1000) making those results at lower points less reliable. 
A method used in the initial testing of our models was angle splitting. We used this when the neural network had a set dimensionality input and we wanted to test different angle numbers’ effect on Mach number and pressure. For example, when our model could only take 4 angles and give 4 angles as an output, we wanted to see how 5 or 6 angles changed our output; however, testing these new angle numbers would require a whole new neural network to be coded. To avoid the time-consuming process that coding a new network would take just to test this angle number theory, we created a combination and separation filter before and after the neural network. These filters would prevent the need for new neural networks to be trained; thus, saving computational resources and time.
	Due to input variable type errors, a function definition script was written for both lists and arrays. The pre-network functions use a while loop to continuously check the array size and act on it when the desired shape (4 angles) is not met. A combination function is called when the array size is smaller than desired, and a separation function is called when the array size is bigger than desired.
	The combination function sorts the numbers in angle input (.sort for list and .argsort for array) then adds the first 2 numbers which are now the lowest (using tuple notation for both list and array: [0] and [1]). This new combined variable is added to the end of the angle input and the first 2 numbers are removed (setting itself to [:2] using tuple notation for list and .delete function for array). This results in a list or array that is now one number shorter without changing the total sum of the input. This process is repeated until the desired length is reached.
	The separation function finds the greatest value in the angle input using index(max()) for lists and argmax for arrays. The return of these functions are assigned to a variable and split into 2 parts: 60 percent and 40 percent. This split is supposed to replicate the logarithmic shape. By implementing a gradual slope instead of an even split, we have a greater chance of getting the shape we solved for in Figure 1. This split percentage was found experimentally, finding where the error was minimized. The 2 new angles are appended to the list or array and the original larger angle is removed using pop for lists and delete for arrays. This results in a list or array that is now one number longer without changing the total sum of the input. This process is also repeated until the desired length is reached.
Due to inherent floating point precision errors in the code, a differential number is added to the last angle of each list and array to ensure the sum is 36. It does this by sorting the outputs of the filters and then checking them with an if statement. If the sum does not equal 36, the difference in actual sum and 36 is calculated. This differential is then added to the last number of the output. This is because the last number has the largest wiggle room due to the logarithmic shape as discussed earlier.
After the inputs of 4 are put through the trained neural network and optimizer, the output is 4 angles. The output is then passed through the filters again to match the size of the desired number of angles. It is worth mentioning that this process is strictly for estimation and testing and should not be perceived as the final product. This is simply one of the processes we used to determine if the amount of angles was a research topic worth implementing in our final code architecture. That being said, this final process of expanding the optimized angle set produced error; however, the results were good enough to pursue a regularly changing angle number design. We observed that, as discussed in the results, as angle number increased, the pressure ratio increased also.
### Conclusion
Our code takes an angle number input, creates a guess array using a logarithmic shape, creates a random dataset of angles and their Mach number and pressure, creates 2 neural networks (Mach and pressure models), trains the models on the generated dataset, uses the trained models in an optimizer, optimizes the guess array, and outputs the optimized angles and their respective Mach number and pressure all in that order. We have built our code this way because we believe that the number of angles desired for the inlet is the most impactful independent variable to be changed in this project. Depending on what the user of this code will choose to constrain, accuracy constraints should be kept at a 3000 sample size, more than 600 epochs, and 17 angles. All other angle sizing testing using this code should also be done with large sample sizes and epochs for desirable results. 
	This project has taught us lots about hypersonic travel and air behavior in these conditions. By manipulating the underbelly geometry of a hypersonic aircraft, we can direct air in such a way that it slows down in velocity without changing in pressure. By increasing the number of angles on the underbelly we can decrease the loss in pressure for every optimized scenario. This benefit does have a quirk in which angles, or dimensions, increase there is a greater error in computation. We simply cannot use too many angles or else our measurements become unpredictable and untrustworthy due to the curse of dimensionality. There is a sweet spot, however, and we have found that to be 7 to 12 angles in our inlet. In these areas we have a small error, increased reliability, and high-pressure ratios. Smaller angle amounts have too small of pressure ratios and any angle amounts greater than 12 have less reliability and higher errors.
