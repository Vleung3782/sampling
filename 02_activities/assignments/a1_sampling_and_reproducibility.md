# ASSIGNMENT: Sampling and Reproducibility in Python

Read the blog post [Contact tracing can give a biased sample of COVID-19 cases](https://andrewwhitby.com/2020/11/24/contact-tracing-biased/) by Andrew Whitby to understand the context and motivation behind the simulation model we will be examining.

Examine the code in `whitby_covid_tracing.py`. Identify all stages at which sampling is occurring in the model. Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.

Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?

Modify the number of repetitions in the simulation to 100 (from the original 1000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.

Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file. The output does not need to match Whitby’s original blogpost/graphs, it just needs to produce the same output when run multiple times

# Author: Victor Leung

```
Please write your explanation here...

PLEASE NOTE: When the whitby_covid_tracing.py file was executed, it ended up in some errors. The whitby_covid_tracing.py was then copied to Jupyter Notebook as the whitby_covid_tracing.ipynb file and executed as instructed by Ciara.


CREATING THE DATAFRAME:
A DataFrame `ppl` is created with 1000 individuals: 200 attending weddings and 800 attending brunches.

Each individual is initially tagged as not infected (`infected = False`) and not traced (`traced = NaN`).

        events = ['wedding'] * 200 + ['brunch'] * 800
        ppl = pd.DataFrame({
            'event': events,
            'infected': False,
            'traced': np.nan  # Initially setting traced status as NaN
        })


ALL STAGES AT WHICH SAMPLING IS OCCURING IN THE MODEL:
1. Infecting a random subset of people (Initial Infection Sampling):
    - Functions used:
        infected_indices = np.random.choice(ppl.index, size=int(len(ppl) * ATTACK_RATE), replace=False)
        ppl.loc[infected_indices, 'infected'] = True

        - np.random.choice():
            - a function from NumPy used to randomly sample elements from an array.

        - size=int(len(ppl) * ATTACK_RATE):
            - len(ppl) returns the total number of rows in the ppl DataFrame, which is 1000 in this case (200 people from weddings + 800 from brunches).
        - ATTACK_RATE is a constant set to 0.10, indicating the model will infect 10% of the ppl dataframe.
        - int(len(ppl) * ATTACK_RATE) = the number of people to be infected = 1000 * 0.10 = 100
        - ppl.loc[ppl['infected'], 'traced'] selects the 'traced' column for the rows where the person is infected.

    - Sampling Frame: All individuals in the `ppl` DataFrame 
    - Sample Size: The number of individuals that are randomly chosen to be infected = 100 
    - Sampling Method: Random sampling without replacement (`replace=False`) 
    - Underlying Distribution: The sampling is done using a uniform distribution over the indices of the `ppl` DataFrame (through `np.random.choice`).


2. Primary Contact Tracing (Tracing Sampling):
    - Functions used:
        ppl.loc[ppl['infected'], 'traced'] = np.random.rand(sum(ppl['infected'])) < TRACE_SUCCESS
    
        - np.random.rand() generates random numbers between 0 and 1 from a uniform distribution. The argument sum(ppl['infected']) specifies the number of random numbers to generate.
        - sum(ppl['infected']) counts how many people in the DataFrame are infected. With 100 people infected, 100 random numbers between 0 and 1.
        - TRACE_SUCCESS = 20% indictes that for each infected individual, there's a 20% chance they will be traced.

    - Sampling Frame: The subset of infected individuals from the previous step.
    - Sample Size: The number of infected individuals is determined by the number of successfully traced infected individuals .
    - Sampling Method: For each infected individual, the function generates a random number (from a uniform distribution) between 0 and 1, and if it’s less than the `TRACE_SUCCESS` rate (20%), that individual is marked as traced. And if it's greater than or equal to 20%, the person is marked as "not traced"(False).
    - Underlying Distribution: Each individual is traced independently based on a Bernoulli distribution with a success probability of 20%.


3. Secondary Contact Tracing (Tracing Based on Event Attendance):
    - Functions used:
        event_trace_counts = ppl[ppl['traced'] == True]['event'].value_counts()
        events_traced = event_trace_counts[event_trace_counts >= SECONDARY_TRACE_THRESHOLD].index
        ppl.loc[ppl['event'].isin(events_traced) & ppl['infected'], 'traced'] = True

        - It selects the individuals who have been traced & then counts the number of traced individuals attended a wedding and the number attended a brunch.
        - The result is a Series with the event type as the index ('wedding', 'brunch') & the count of traced individuals for each event type as the values.
        - If an event (wedding or brunch) has at least 2 traced cases, all infected individuals at that event are traced (secondary contact tracing) 
        i.e. SECONDARY_TRACE_THRESHOLD = 2, the minimum number of traced cases required in an event to trigger secondary contact tracing.

    
    - Sampling Frame: The individuals who have been successfully traced in the primary tracing step.
    - Sample Size: The number of events that have enough traced individuals to trigger secondary tracing. The SECONDARY_TRACE_THRESHOLD is set to 2.
    - Sampling Method: It involves determining whether each event (wedding or brunch) has at least 2 traced individuals. If an event meets the SECONDARY_TRACE_THRESHOLD of 2 of, all infected individuals who attended that event are marked as traced.
    - Underlying Distribution: This step is deterministic, as it is based on the number of individuals traced at each event, and does not rely on randomness or probability. Instead, it is entirely determined by the SECONDARY_TRACE_THRESHOLD. The rule of sampling itself determines the outcome, and the "distribution" of outcomes is just a single point (there's no spread or variability).


DOES THE CODE (from the Python script file whitby_covid_tracing.py) APPEAR TO REPRODUCE THE GRAPHS FROM THE ORIGINAL BLOG POST?
    
    The script produces two histograms: 
        - Infections from Weddings (Blue): The proportion of infections attributed to weddings.
        - Traced to Weddings (Red): The proportion of traced cases attributed to weddings.
    
    Unexpectedly, the histograms from the execution of the Python script file do not seem to reproduce the histograms from the original blog post. Below are the observations:
        - In the original blog post, the mean observed proportion is around double the true proportion and the modal observed value is even higher. The results in Whitby's blog post showed that while weddings account for a smaller proportion of infections, they account for a larger proportion of traced cases due to the bias introduced by contact tracing. 
        - From the execution of the Python script, there is significant overlap of the blue & red plots, in sharp contrast to Whitby's original blog post. 

    



COMMENT ON THE REPRODUCIBILITY OF THE RESULTS AFTER MODIFYING THE NUMBER OF REPETITIONS IN THE SIMULATION TO 100 (from the original 1000):
    After reducing the number of repetitions in the simulation to 100 (using the code below) and runnning the script multiple times, there is greater variability in the output plots.

        results = [simulate_event(m) for m in range(100)]

    This is expected as the standard error is proportional to square root of the number of simulations, consistent with the Central Limit Theorem. 



CHANGES MADE TO THE CODE SO THAT IT IS REPRODUCIBLE:
    To ensure reproducibility, a random seed was established. The following line of code has been added to the beginning of the Python script:

        np.random.seed(911)

    "911" is an arbitrary number to ensure the same sequence of random numbers are generated each time the script is executed. 

    As a result, the output plots are identical each time the script is run with the same proportions of infections and traces across runs.

```


## Criteria

|Criteria|Complete|Incomplete|
|--------|----|----|
|Altercation of the code|The code changes made, made it reproducible.|The code is still not reproducible.|
|Description of changes|The author explained the reasonings for the changes made well.|The author did not explain the reasonings for the changes made well.|

## Submission Information

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

### Submission Parameters:
* Submission Due Date: `23:59 - 16/02/2025`
* The branch name for your repo should be: `assignment-1`
* What to submit for this assignment:
    * This markdown file (a1_sampling_and_reproducibility.md) should be populated.
    * The `whitby_covid_tracing.py` should be changed.
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sampling/pull/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-1`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via the help channel in Slack. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.
