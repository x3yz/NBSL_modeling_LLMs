# NBSL for Modeling Interacting LLMs
$s_1:=$\"\[Pulp Fiction, Forrest Gump, Jurrasic Park\]\"  
$s_2:=$\"[Pulp Fiction, Forrest Gump, Fight Club\]\"
$\theta^*=1*
## Download and run data
For Experiment 1 extract [ml-20m](https://grouplens.org/datasets/movielens/20m/) to \"Experiment1/data/\" ; for Experiment 2 extract [ml-32m](https://grouplens.org/datasets/movielens/32m/) to \"Experiment2/data/\"
## Experiment description
In the first experiment, after getting the tags associated with each of the three selected movies $s_1$, we first reindex the tags, the result can be found (in order of the indexes) in the file \"Experiment1/data/originaltags.pkl\". We then manually create a split of all the tags into six related categories and randomly further split each of the six groups in half with the numpy random seed 42. We then get tags associated with every movie.  The result is an array *moviegroups* with $K_1=12$ groups that can be found in the file \"Experiment1/data/Moviegroups.pkl" file in the repository. If the correct movie is $\theta$, *moviegroups*(${\theta, k}$) fixes the tags that the agent $k$ will be able to receive during the experiment.  
We then, using the random seed 42 and numpy.random library, get a random permutation of the first $20$ numbers, *arr=shuffled_times*. Then at time $i$ agent $k$ in both the NBSL algorithm and LLM algorithm receives the tag with index *moviegroups*($\theta^\*,k,arr_i$). See the notebooks in \"Experiment1\" for more details.  
In the second experiment, after getting the tags associated with each of $s_2$, we first split the tags associated with $\theta^{\*} =1$ and then pick $300$ of these new split tags randomly, leading to the array \"Experiment2/data/sampled_split_tags.pkl\". We then unite $140$ of these new tags with the original tags, and get the array named *extended_tags*, \"Experiment2/data/extended_tags.pkl\". Then we randomly split the **original** tags up into 23 groups, so that the groups associated with the movie $\theta^\*$ are about equal, leading to the arrays *Moviegroups* and *Movietags*.  
We then, again with random seeds 42, start sampling $15$ of these tags for every agent, according to the formula: 25% of the time we take a tag from the new shuffled tags, 55% of the time we tage a tag from the correct *moviegroups*($\theta^*$ , k), and 20% of the time from the wrong movie. This is done in order to make learning the correct movie harder, everything can be viewed in more detail in the file \"Experiment2/RunNBSL&GetClueOrder.ipynb\". Then this choice is saved in an array \"Experiment2/data/clueids\".  
In order to run the simulation, it is enough to just sample the next tag from $\text{clueids}_{i, k}$. 
## Prompts used
1) Prompt descriptions: likelihood collection for experiment $j$  
   **Header**:  Presume that any of the three movies is initially equally likely. Then a single tag is given, related to the correct movie. Return ONLY three space-separated decimal numbers. NO explanations, NO formatting.  
   **Content**: Given the tag: \"{clue}\", what is your belief over the three classes $s_j$? Return ONLY EXACTLY three space-separated decimal numbers. NO explanations, NO text, NO formatting.  
2) Prompt descriptions: Self-learning for experiment $j$ at time $i$ and agent $k$  
   **Header**: You are participating in a movie guessing game with three possible movies.  Your previous belief is given; as well as a clue related to the correct movie. Your task is to output your new belief probabilites. Return ONLY three space-separated decimal numbers. NO explanations, NO text, NO formatting.  
   **Content**: New clue: \"clue\". Your own belief from the previous round over $s_i$ was: $\mu_{k, i-1}$.  Based on the new clue and your previous belief, what is your belief over the classes $s_j$? Return ONLY EXACTLY three space-separated decimal numbers. NO explanations, NO text, NO formatting   
3) Prompt descriptions: Combination step for experiment $j$ at time $i$ and agent $k$  
   **Header**: You are participating in a movie guessing game with three possible movies. Your previous belief and your neighbors' beliefs are given. Your neighbors have seen different clues than you. Your task is to output your new belief probabilites. Return ONLY three space-separated decimal numbers. NO explanations, NO text, NO formatting.  
   **Content**: Your own belief from the previous round over $s_j$ was: $\psi_{k, i}$. Your neighbors' beliefs for $s_j$ are  
   *Here output all neighbors in the form:* Neighbor $m$: $\psi_{m, i}$   
   Based on the provided information, what is your current probability belief over $s_j$? Return ONLY EXACTLY three space-separated decimal numbers. NO explanations, NO text, NO formatting.  
