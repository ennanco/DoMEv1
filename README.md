# DoMEv1

This project contains the source code of the first version of the DoME algorithm for Symbolic Regression, written in Julia. The aim of this code is to be able to repeat the experiments described in the paper. For this reason, this code will not be updated. Instead, a new project will be created with the DoME source code that will incorporate the updates. However, this code is fully functional. Feel free to use this source code to perform your experiments. However, if any publication is generated through this system, please add a citation to the following paper:

# How to use DoME

The easiest way to wun DoME is by calling the function dome. Here is an example of use (available on the file "exampleGithub.jl"), in which only the main hyperparameters are set:

	using FileIO
	using DelimitedFiles
	
	# Load the dataset and create a matrix with the inputs and a vector for the targets
	dataset = DelimitedFiles.readdlm("datasets/561_cpu.tsv");
	inputs  = Float64.(dataset[2:end, 1:end-1]);
	targets = Float64.(dataset[2:end, end]);

	# Load the DoME system
	include("DoME.jl");
	# Run DoME with these parameters
	(trainingMSE, validationMSE, testMSE, bestTree) = dome(inputs, targets;
	   minimumReductionMSE = 1e-6,
	   maximumNodes = 30,
	   strategy = StrategyExhaustive,
	   showText = true
	);
	# Write the expression on screen
	println("Best expression found: ", string(bestTree));
	println("Best expression found (written in Latex): ", latexString(bestTree));
	# If you want to rename the variable names, one of the easiest way is to do something like:
	expression = string(bestTree);
	expression = replace(expression, "X1" => "vendor");
	expression = replace(expression, "X2" => "MYCT");
	expression = replace(expression, "X3" => "MMIN");
	expression = replace(expression, "X4" => "MMAX");
	expression = replace(expression, "X5" => "CACH");
	expression = replace(expression, "X6" => "CHMIN");
	expression = replace(expression, "X7" => "CHMAX");
	println("Best expression found (with the real names of the variables): ", expression);

When calling the function dome, inputs is a NxP matrix of real numbers, and targets is a N-length vector or real numbers (N: number of instances, P: number of variables). Inputs and targets can have Float32 or Float64 values; however, since many constants are generated during the run of the algorithm, it is recommended to use Float64 to have the highest precision. Also, the elements of both inputs and targets must have the same type (Float32 or Float64). The parameters minimumReductionMSE, maximumNodes and strategy are the 3 hyperparameters described in the paper.

The declaration of this function is the following, with the whole set of parameters and their default values:

	function dome(inputs::Array{<:Real,2}, targets::Array{<:Real,1};
	    # Each instance in inputs is in a row or in a column
	    dataInRows          ::Bool           = true,
	    # Hyperparameters of the algorithm
	    minimumReductionMSE ::Real           = 1e-6,
	    maximumNodes        ::Int64          = 50 ,
	    strategy            ::Function       = StrategyExhaustive ,
	    # Other hyperparameter that the user might find useful
	    maximumHeight       ::Real           = Inf ,
	    # Stopping criteria
	    goalMSE             ::Real           = 0 ,
	    maxIterations       ::Real           = Inf ,
	    # Indices of the instances used for validation and test (validation has not been tested and may fail)
	    validationIndices   ::Array{Int64,1} = Int64[],
	    testIndices         ::Array{Int64,1} = Int64[],
	    # Validation and test ratios (these parameters have not been tested and may fail)
	    validationRatio     ::Real           = 0. ,
	    testRatio           ::Real           = 0. ,
	    # If you want to see the iterations on screen. This makes the execution slower
	    showText            ::Bool           = false ,
	    # This parameter was used only for development. If it is set to true, the execution becomes much slower
	    checkForErrors      ::Bool           = false
	)

The description of these parameters is the following:

	dataInRows -> allows the input matrix to have dimensions NxP when it is set to true (by default) or PxN when it is false (N: number of instances).
	minimumReductionMSE -> A search is found to be successful if the reduction in MSE is positive and higher than the previous MSE value multiplied by this parameter.
	maximumNodes -> maximum number of nodes in the tree.
	strategy -> the strategy used to select which searches are going to be performed on which nodes. The 4 strategies described in the paper are available, with names StrategyExhaustive (by default), StrategyExhaustiveWithConstantOptimization, StrategySelectiveWithConstantOptimization and StrategySelective. They are also called Strategy1, Strategy2, Strategy3, Strategy4 respectively as used in the paper.
	maximumHeight -> maximum height of the tree. As explained in the paper, this parameter is not recommended to be used in the experiments.
	goalMSE -> if the algorithm reaches this MSE value in training, the iterative process is stopped.
	maxIterations -> maximum number of iterations to be performed.
	validationIndices -> allows to split the dataset by separating some instances to perform the validation, specifying which ones will be used for validation.
	testIndices -> allows to split the dataset by separating some instances to perform the test, specifying which ones will be used for test.
	validationRatio ->   allows to split the dataset by separating some random instances to perform the validation, specifying the ratio used for validation.
	testRatio ->   allows to split the dataset by separating some random instances to perform the test, specifying the ratio used for test.
	showText -> if it is set to true, on each iteration some text (iteration number, best tree, MSE in training and test) is shown.
	checkForErrors -> this parameter was used only for debugging, to easily find bugs in the source code. Therefore, it is best to leave it as false.

As it can be sen, this function allows the definition of a validation set. However, it was not used in the experiments of the paper and thus this part of the code may have errors.

To run this, you need to have the following files in the same folder:

	- DoME.jl -> Main algorithm, with the searches and the 4 strategies.
	- Tree.jl -> Tree structure creation and some useful functions to operate.
	- Equation.jl -> Management of the equations of each node, and calculation of the best constant for each node.
	- NodePool.jl -> Struct for storing nodes to be used in the tree. This struct is filled with variables; however, in the future other nodes (and subtrees) could be stored on it.

An alternative way to run DoME is by creating a DoME struct and calling the function Step! for each iteration. This is automatically done by the previous way to run DoME. To eun DoME, only the packages Statistics and Random are needed.

# How to repeat the experiments described in the paper

To obtain the values of tables 1 and 3 in the paper, run the file examplePaper.jl

To obtain the values of Table 4 in the paper (Newton's law of universal gravitation), run the file experimentNewton.jl

To repeat the experiments describen in section 4, create a folder named "datasets" and store the datasets from PMLB used in the paper. When running the experiments, a folder named "results" will be created. These experiments were performed with a Slurm job scheduling system. The following files were used to perform the experiments:

	- usefulFunctions.jl -> Defines the hyperparameters to be used in the experiments (values of maximum number of nodes, minimum MSE reduction, number of folds), and has useful functions that allows the load of tsv files, create crossvalidation indices, check if an experiment is already finished, etc. It uses the packages FileIO, DelimitedFiles and Random.
	- createScripts.jl -> Creates the scripts for the Slurm schedule. For each dataset, 4 scripts are created, one for each strategy. Also, two additional scripts are created, names executeAll and cancelAll, that allows the submission and cancel of all of the jobs.
	- experimentsDoME.jl -> Performs the experiments on the dataset and strategy specified on the command prompt. It uses the package JLD2 to store the results.
	- readResults.jl -> Returns the data for tables 5, 6, 7, and 8, and figures 5, 6, 7, and 8 from those datasets whose experiments (in the 4 strategies) have finished. The figures are stored as pdf files in the results folder. It uses the packages FileIO, JLD2, Statistics, Printf, StatsPlots, CriticalDifferenceDiagrams, DataFrames, and PGFPlots.

Even these files are written for a Slurm schedule system, the experiments can be performed on a single computer by just modifying and running experimentsDoME.jl

# How to define your own strategy

Strategies are based on calling the functions PerformSearches! and OptimizeConstants!

The function PerformSearches! allows to specify in the call which nodes are going to be used on each search. To do this, this function has keyword parameters for each search, and in each one the user can specify type of the nodes in which this search will take place. These types are Terminal, Variable, Constant, and NonTerminal. Also, the types Any and Nothing can be used to specify that a search will be performed on all of the nodes, or in none of them respectively. The declaration of this function is the following:

	function PerformSearches!(obj::DoME;
	   whichNodesPerformConstantSearch        ::Union{DataType,Union} = Nothing ,
	   whichNodesPerformVariableSearch        ::Union{DataType,Union} = Nothing ,
	   whichNodesPerformConstantVariableSearch::Union{DataType,Union} = Nothing ,
	   performConstantExpressionSearch        ::Bool = false)

Note that constant-expression search receives a boolean value, because this search is only performed on non-terminal nodes.

This function returns a Boolean value: if it is true, a search has been succesful, otherwise no search was succesful. The strategy function to be defined should also return a Boolean value, with the same interpretation.

An example is the Exhaustive strategy, in which the searches are performed on all of the nodes of the tree:

	function StrategyExhaustive(obj::DoME)
	   changeDone = PerformSearches!(obj;
	      whichNodesPerformConstantSearch=Any ,
	      whichNodesPerformVariableSearch=Any ,
	      whichNodesPerformConstantVariableSearch=Any ,
	      performConstantExpressionSearch=true);
	   return changeDone;
	end;

Another example is the Selective strategy, that performs the searches performs searches sequentially, moving on to the next one only if the previous one has been unsuccessful:

	function StrategySelective(obj::DoME)
	   changeDone =               PerformSearches!(obj; whichNodesPerformConstantSearch=Constant);
	   # Variable search only on constants
	   changeDone = changeDone || PerformSearches!(obj; whichNodesPerformVariableSearch=Constant);
	   changeDone = changeDone || PerformSearches!(obj; performConstantExpressionSearch=true);
	   # Constant-variable search only on terminals
	   changeDone = changeDone || PerformSearches!(obj; whichNodesPerformConstantVariableSearch=Union{Constant,Variable});
	   if (!changeDone)
	      # Constant search on variables and non-terminals, variable seach on variables and non-terminals, and constant-variable search on non-terminals
	      changeDone = PerformSearches!(obj;
	         whichNodesPerformConstantSearch=Union{Variable,NonTerminal} ,
	         whichNodesPerformVariableSearch=Union{Variable,NonTerminal} ,
	         whichNodesPerformConstantVariableSearch=NonTerminal );
	   end;
	   return changeDone;
	end;

Also, inside the strategy the function OptimizeConstants! can be called. This function performs the constant optimization process as described in the paper, and returns a boolean value with the same interpretation. The strategies corresponding to Exhaustive and Selective but with constant optimisation are as follows:

	function StrategyExhaustiveWithConstantOptimization(obj::DoME)
	   changeDone = PerformSearches!(obj;
	      whichNodesPerformConstantSearch=Union{Variable,NonTerminal} ,
	      whichNodesPerformVariableSearch=Any ,
	      whichNodesPerformConstantVariableSearch=Any ,
	      performConstantExpressionSearch=true);
	   changeDone && OptimizeConstants(obj);
	   return changeDone;
	end;

	function StrategySelectiveWithConstantOptimization(obj::DoME)
	   # Variable search only on constants
	   changeDone =               PerformSearches!(obj; whichNodesPerformVariableSearch=Constant);
	   changeDone = changeDone || PerformSearches!(obj; performConstantExpressionSearch=true);
	   # Constant-variable search only on terminals
	   changeDone = changeDone || PerformSearches!(obj; whichNodesPerformConstantVariableSearch=Union{Constant,Variable});
	   if (!changeDone)
	      # Constant search on variables and non-terminals, variable seach on variables and non-terminals, and constant-variable search on non-terminals
	      changeDone = PerformSearches!(obj;
	         whichNodesPerformConstantSearch=Union{Variable,NonTerminal} ,
		 whichNodesPerformVariableSearch=Union{Variable,NonTerminal} ,
		 whichNodesPerformConstantVariableSearch=NonTerminal );
	   end;
	   changeDone && OptimizeConstants(obj);
	   return changeDone;
	end;

These four strategies are provided in the file DoME.jl and are available for use.
