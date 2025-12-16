
# Agents

This is a simulation of agents coming to consensus minimizing the laplacian value of a graph.

The implementation is based on synthesis of networks Chapter - Section 11.4 from ["Graph Theoretic methods in multiagent networks"](https://press.princeton.edu/books/hardcover/9780691140612/graph-theoretic-methods-in-multiagent-networks?srsltid=AfmBOorOlSMQ-ixZBA_9Zth86rRCyIes-ysjBLg0Pg5OiiR3YBaQ_1dD)


#### Excerpt from the book

The excerpt from the section is as follows.


Consider n elements which are the vertices of the graph and the edges determined by the relative positions.

We define graphs of order n ( V = n vertices ) with edge set E ( E is the edge set with i, j vertices )  with no loops.

a weighting function 

$w : \mathbf{R}^3 \times \mathbf{R}^3 \rightarrow \mathbf{R}_{+}$

Each element is 3 dimensional and the final weight is a single positive real value determined by the euclidean distance


$w_ij = f(||x_i - x_j||)$

The function f should be 1 if the distance is less than a certain distance and then rapidly drop to 0 as distance increases. ( I think of this like a modified, reflected and shifted sharp sigmoid function ). This is similar to a wireless network signal.

With this setup, let's look at the configuration problem:

$\Lambda = max_x \lambda_2(G, x)$

is the optimized value, where lambda is the second smallest eigenvalue for the laplacian. This element tells us how connected the network is and greater the value, the graph is very connected. Now in this function, x is a parameter and x is also the vector of all node positions..This just means that the node positions should maximize this particular value - the second largest value of the laplacian, which means achieve maximum connectivity.

Here x is any node of the graph we defined above

Next we define the graph Laplacian.

from the book,

![laplacian](shots/laplacian.png)

In words, this just means that, for all off diagonals, the weight is the weight from our defined weight function, and for all the diagonal entries, the weights are the sum of all the weights of other elements. This is the exact weighted version of the unweighted laplacian matrix.

Now we also impose a constrain that all objects should stay atleast $\rho$ distance apart. This is to make sure that the nodes don't collapse into a point to essentially blow up the laplacian to infinity.

Now straight out of the bat, this seems like an intractable problem with no way to formulate as a convex optimization problem subsequently a solution dependent on interior point methods ( THese are methods where instead of traversing the optimal boundry, we traverse interior of the region bounds )

But, our problem is somewhat alleviated, as the smallest eigenvalue is always 0, with the associated eigenvector always containing unit entries. 

Even then, the relative distances problem is nonlinear and need non convex optimization. Here the author proposes a semidefinite programming based approach ( an approach where the non negativity constraints become semidefiniteness on matrices - basically an optimization on matrix variables ). 


We also need to define f before proceeding and the choice of f depends on applications and numerical conditions. in our case, f takes the form

$f(d_{ij}) = /epsilon^{(\rho_1 - d_ij)/(\rho_1 - \rho_2)}$


This function is clever - the exponent makes it such that at $d = \rho_2$, value is $1$ and at $d = \rho_1$ value is $\epsilon$
The curve stays constant under the threshold distance, and then falls exponentially to a small value greater than the threshold. also makes the function smooth and differntiable leading to a stable algorithm for our experimentation.


Interesting part: Now the authors show a bunch of proofs. In summary the proof shows that the $\lambda_2 $ values can be expressed using matrix inequalities. They achieve this by introducing an orthonormal matrix P, whose columns live in subspace of $1^\bot$ which is the matrix with columns orthonormal to all 1 vector. This works because we already know that the first $\lambda$ of the laplacian is always 0, and the eigenvector is all ones.

so P is the orthonormal vector that actually contains the eigenvalues $\lambda_2, \lambda_3 ...$


So we have a physical distance constraint and the spectral constraint.

$P^T L(G) P \geq \gamma I_{n-1}$

We have now formulated our problem as a semidefinite programming problem! that depends on $\gamma$ which is a single number and we can now perform convex optimization on this. Optimizing $\lambda_2$ has now been translated into a problem optimizing $\gamma$


### Optimization

Now moving to the optimization part, we can differentiate the distance constraint wrt to time.

$d_{ij}(t) = ||x_i - x_j||^2$

This is the matrix distance, differentiating

and ends up being 

$d_{ij}'(t) = 2(x_i'(t) - x_j'(t))^T (x_i(t) - x_j(t)) = d_{ij}'(t)$

we employ eulers discretization method and $\Delta t$ as the sampling, 

$x(t) \rightarrow x(k) , x(k+1) - x(k) \approx x'(t) \Delta t$

so 

$d_{ij}'(t) = d_{ij}(k+1) + d_{ij}(k) = 2(x_i(k+1) - x_j(k+1))^T(x_i(k) - x_j(k)$


similarly, state dependent laplacian is discretized the same way wrt to time and then with Euler discretization, ending up in,

$w_{ij}(k+1) = w_{ij}(k) - \epsilon^{(\rho_1 - d_{ij} (k) ) / (\rho_1 - \rho_2)}(d_{ij}(k+1) - d_{ij}(k))$


Including the constraints on the weights, we arrive at a iterative step for the Semidefinite programming, starting at time k = 0 and iteration.


The proposed algorithm approximates a non-convex problem with linear approximation. This might introduce potential inconsistency between position and distance vectors. Here by adding the two convex constraints, we always obtain consistency in position and distance at each iteration. we can also further reduce inconsistency by updating x(k) after the calculation of x(k)


Okay, in summary, we now know that optimization has become a problem of optimizing $\gamma$ under the constraints, $\lambda_2(G,x) >= \gamma$
and at equilibrium $\gamma = \lambda_2$ at which point we have reached the solution.


Now we just need to implement this!


In summary, here's how the optimization is going to go.

### Initializations

Initial Locations:
x - random locations on the environment

The weighting function:
$f(d_{ij}) = \epsilon^{(\rho_1 - d_ij)/(\rho_1 - \rho_2)}$

choose $\gamma$ under constraint $P^T L(G) P \geq \gamma I_{n-1}$


