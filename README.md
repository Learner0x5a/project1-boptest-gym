# BOPTEST-Gym

BOPTESTS-Gym is the [OpenAI-Gym](https://gym.openai.com/) environment for the [BOPTEST](https://github.com/ibpsa/project1-boptest) framework. This repository accommodates the BOPTEST API to the OpenAI-Gym convention in order to facilitate the implementation, assessment and benchmarking of reinforcement learning (RL) algorithms for their application in building energy management. RL algorithms from the [Stable-Baselines 3](https://github.com/DLR-RM/stable-baselines3) repository are used to exemplify and test this framework. 

The environment is described in [this paper](https://www.researchgate.net/publication/354386346_An_OpenAI-Gym_environment_for_the_Building_Optimization_Testing_BOPTEST_framework). 

## Structure
- `boptestGymEnv.py` contains the core functionality of this Gym environment.
- `environment.yml` contains the dependencies required to run this software. 
- `/examples` contains prototype code for the interaction of RL algorithms with an emulator building model from BOPTEST. 
- `/testing` contains code for unit testing of this software. 

## Quick-Start (using BOPTEST-Service)
BOPTEST-Service allows to directly access BOPTEST test cases in the cloud, without the need to run it locally. Interacting with BOPTEST-Service requires less configuration effort but is considerably slower because of the communication overhead between the agent and the test case running in the cloud. Use this approach when you want to quickly check out the functionality of this repository. 

1) Create a conda environment from the `environment.yml` file provided (instructions [here](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file)). 
2) Check out the `boptest-gym-service` branch and run the example below that uses the [Bestest hydronic case with a heat-pump](https://github.com/ibpsa/project1-boptest/tree/master/testcases/bestest_hydronic_heat_pump) and the [DQN algorithm](https://stable-baselines3.readthedocs.io/en/master/modules/dqn.html) from Stable-Baselines: 

```python
from boptestGymEnv import BoptestGymEnv, NormalizedObservationWrapper, DiscretizedActionWrapper
from stable_baselines3 import DQN

# url for the BOPTEST service. 
url = 'https://api.boptest.net' 

# Decide the state-action space of your test case
env = BoptestGymEnv(
        url                  = url,
        testcase             = 'bestest_hydronic_heat_pump',
        actions              = ['oveHeaPumY_u'],
        observations         = {'time':(0,604800),
                                'reaTZon_y':(280.,310.),
                                'TDryBul':(265,303),
                                'HDirNor':(0,862),
                                'InternalGainsRad[1]':(0,219),
                                'PriceElectricPowerHighlyDynamic':(-0.4,0.4),
                                'LowerSetp[1]':(280.,310.),
                                'UpperSetp[1]':(280.,310.)}, 
        predictive_period    = 24*3600, 
        regressive_period    = 6*3600, 
        random_start_time    = True,
        max_episode_length   = 24*3600,
        warmup_period        = 24*3600,
        step_period          = 3600)

# Normalize observations and discretize action space
env = NormalizedObservationWrapper(env)
env = DiscretizedActionWrapper(env,n_bins_act=10)

# Instantiate an RL agent
model = DQN('MlpPolicy', env, verbose=1, gamma=0.99,
            learning_rate=5e-4, batch_size=24, 
            buffer_size=365*24, learning_starts=24, train_freq=1)

# Main training loop
model.learn(total_timesteps=10)

# Loop for one episode of experience (one day)
done = False
obs, _ = env.reset()
while not done:
  action, _ = model.predict(obs, deterministic=True) 
  obs,reward,terminated,truncated,info = env.step(action)
  done = (terminated or truncated)

# Obtain KPIs for evaluation
env.get_kpis()

```

## Quick-Start (running BOPTEST locally)
Running BOPTEST locally is substantially faster

1) Create a conda environment from the `environment.yml` file provided (instructions [here](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file)). 
2) Run a BOPTEST case with the building emulator model to be controlled (instructions [here](https://github.com/ibpsa/project1-boptest/blob/master/README.md)).  
3) Check out the `master` branch of this repository and run the example above replacing the url to be `url = 'http://127.0.0.1:5000'` and avoiding the `testcase` argument to the `BoptestGymEnv` class. 


## Citing the project

Please use the following reference if you used this repository for your research.

```
@inproceedings{boptestgym2021,
	author = {Javier Arroyo and Carlo Manna and Fred Spiessens and Lieve Helsen},
	title = {{An OpenAI-Gym environment for the Building Optimization Testing (BOPTEST) framework}},
	year = {2021},
	month = {September},
	booktitle = {Proceedings of the 17th IBPSA Conference},
	address = {Bruges, Belgium},
}

```



