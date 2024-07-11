# Optimization-of-Energy-Absorbent-Shells-discontinuities-Size-and-shape-in-Julia
In this research, the energy-absorbing behavior of some thin wall shell will be investigated by numerical solution in finite element method and in Abaqus software, and the size and shape of the absorber will be Optimized with proper discontinuities.
In this research, the energy-absorbing behavior of a thin wall with an aluminum rec-tangular section will be investigated by numerical solution in finite element method and in Abaqus software, and to improve its energy absorption parameters, discontinuities will be created on the geometry, which regularized the folding pattern. The research approach is to improve the energy absorption behavior without fundamentally chang-ing the geometry. Discontinuities with different geometrical shapes, sizes, and numbers will be placed in different places in the model.
For the optimization, considering the characteristics of the energy absorber, the SEA and CFE of the energy absorber are selected as the crashworthiness optimization objective functions. Then, the discontinuity size, number, and position are taken as the design variables.
In this report we assume the number of variables to be two because of the complexity of polynomial response surface that provide an objective function for problem. This will explain in nex section. So the selected variables is diameter of first discontinuity on front wall(d_f1) and height of first discontinuity on front wall(h_f1). That means we have a discontinuity and we want to optimize the diameter and position of the discontinuity.

The code of optimization in Julia:

using Optim
using Random

# Seed the random number generator for consistent results
Random.seed!(147)

# Define the objective function
function objval_penalty(x::Vector{Float64})
    # Original objective function
    F1 =  0.8037 -12.4377*x[1] + 3.072*x[2] + 1077.01*x[1]^2 - 43.4791*x[1]*x[2] -25.3044*x[2]^2  -25727.2826*x[1]^3 + 53.2505*x[1]^2*x[2]  -2.5304*x[1]*x[2]^2 + 66.9916*x[2]^3 + 216498.9582*x[1]^4  -1286.3725*x[1]^3*x[2] + 2.6575*x[1]^2*x[2]^2 + 0.8291*x[1]*x[2]^3 + 26.1695*x[2]^4
    F2 = 0.9652 -8.3753*x[1]+ 1.4082*x[2] + 615.092*x[1]^2  -11.5786*x[1]*x[2]  -8.6511*x[1]x[2]^2  -16206.5127*x[1]^3 + 30.5984*x[1]^2*x[2]  -0.701*x[1]*x[2]^2 + 16.6612*x[2]^3 + 147370.2355*x[1]^4 + -810.3278*x[1]^3*x[2] + 1.5282*x[1]^2*x[2]^2 + 0.2043*x[1]*x[2]^3 + 6.6066*x[2]^4
	
    # Nonlinear constraints
    Cineq, Ceq = nonlcon(x)
    
    # Penalty terms (using broadcasting for element-wise operations)
    penalty = sum(Cineq.^2) + sum(Ceq.^2)
    
    # Objective function with penalty
    return F1 + F2 +  penalty
end

# Nonlinear constraint function
function nonlcon(x::Vector{Float64})

    
    Cineq = Float64[]
    Ceq = Float64[] # Empty array for equality constraints
    
    return Cineq, Ceq
end


# Variable bounds
lb = [0.005, 0.04]
ub = [0.06, 0.35]

# Define your initial point (within the bounds)
initial_point = [0.05, 0.05]

# PSO options
opt = ParticleSwarm(lower=lb, upper=ub, n_particles=200)

# Run PSO multiple times and average results
num_runs = 10 # Number of optimization runs
best_solutions = []
best_objective_values = []

for _ in 1:num_runs
    result = optimize(objval_penalty, initial_point, opt, Optim.Options(iterations=2000))
    push!(best_solutions, result.minimizer)
    push!(best_objective_values, result.minimum)
end

# Calculate and print average results
average_solution = sum(best_solutions) / num_runs
average_objective_value = sum(best_objective_values) / num_runs

println("Average Optimal Solution (over $num_runs runs):")
println(average_solution)
println("Average Objective Function Value:")
println(average_objective_value)

