- Machine learning algorithms/programs are focused on self improvement at some task
- ML problems can be broken down into:
	- A task
	- A performance metric
	- A source for training data
- The exact definitions of the above guide the utility of the problem
- With the problem at hand, there's a theoretical space of hypotheses that now represent all the possible ways to complete the task, with a "best" hypothesis being whatever maximizes the performance metric
- The target function is a theoretical goal to aim for in order to find the best performing hypothesis. ML problems are now in a sense a search problem to either find or approximate this target function
- In reality, ML covers hypothesis spaces where it is practically impossible to search exhaustively, hence the need for the self improvement to help with the efficiency

### Exercises
- 1.3: Prove that the LMS weight update rule described in this chapter performs a gradient descent to minimize the squared error. In particular, define the squared error $E$ as in the text. Now calculate the derivative of $E$ with respect to the weight $w_i$, assuming the $\hat{V}(b)$ is a linear function as defined in the text. Gradient descent is achieved by updating each weight in proportion to $-\partial E/\partial w_i$. Therefore, you must show that the LMS training rule alters weights in this proportion for the Experiment Generator module of Figure 1.2.
![[Exercise1.3.png]]