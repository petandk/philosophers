*This project has been created as part of the 42 curriculum by rmanzana*

# Philosophers

## Description

**Philosophers** is a simulation of the classic **dining philosophers problem**, written in C with **threads and mutexes**.

A number of philosophers sit at a round table with a bowl of spaghetti in the middle. There is one fork between each pair of neighbours, and a philosopher needs **both** forks next to them to eat. They cycle through **eating, sleeping and thinking**, and if one of them goes too long without eating, they die and the simulation stops.

The goal of the project is to learn the basics of concurrent programming: creating threads, protecting shared data with mutexes, and avoiding **data races**, **deadlocks** and **starvation**.

Each philosopher is a thread, and each fork is a mutex.

## How it works

### Structure

- **`t_shared`**: the data every thread can see — the timing parameters, the array of fork mutexes, the start time, and flags (someone died / simulation still running), each protected by its own mutex.
- **`t_philo`**: one per philosopher — its id, pointers to its left and right forks, its last meal time and meal counter (protected by a per-philosopher mutex).
- **`t_table`**: puts it all together, with the array of philosophers and their threads.

### The philosopher routine

1. **Synchronised start.** A `ready_mutex` is locked before the threads are created. Every philosopher blocks on it until all threads exist and the start time is set, so nobody starts early.
2. **Eat.** Take two forks, update `last_meal` and the meal counter, eat for `time_to_eat`, release the forks.
3. **Sleep** for `time_to_sleep`.
4. **Think**, then loop until the simulation ends.

### Avoiding deadlocks

If every philosopher took their left fork at the same time, they would all wait forever for the right one. To prevent this, **odd philosophers take their left fork first, and even philosophers take their right fork first**. This breaks the circular wait, so at least one philosopher can always get both forks.

A single philosopher has only one fork: they take it, wait `time_to_die` and die, as expected.

### The monitor ("croupier")

The main thread acts as a monitor. Every millisecond it checks each philosopher:

- If the time since their last meal is greater than `time_to_die`, it sets the `philo_died` flag, stops the simulation and prints `died`.
- If the optional `number_of_times_each_philosopher_must_eat` was given and every philosopher has eaten that many times, it stops the simulation.

Philosophers check the "still running" flag at every loop iteration and exit cleanly. Then all threads are joined and every mutex is destroyed and freed.

### Avoiding data races

Every variable shared between threads is read and written **only while holding its mutex**: the last meal time and meal counter, the death flag, the "keep eating" flag, and standard output (`print_mutex`), so log lines are never mixed or printed after a death.

## Instructions

### Compilation

```bash
make          # builds the philo executable
make clean    # removes object files
make fclean   # removes object files and the executable
make re       # rebuilds everything
```

### Usage

```bash
./philo number_of_philosophers time_to_die time_to_eat time_to_sleep [number_of_times_each_philosopher_must_eat]
```

All times are in milliseconds. Arguments must be positive integers different from zero.

```bash
./philo 1 800 200 200        # the philosopher dies (only one fork)
./philo 5 800 200 200        # nobody should die
./philo 5 800 200 200 7      # stops when everyone has eaten 7 times
./philo 4 410 200 200        # nobody should die
./philo 4 310 200 100        # a philosopher should die
```

### Output

Every state change is logged with a timestamp (in ms since the start) and the philosopher's number:

```
0 1 has taken a fork
0 1 has taken a fork
0 1 is eating
200 1 is sleeping
400 1 is thinking
...
```

## Resources

- [Dining philosophers problem — Wikipedia](https://en.wikipedia.org/wiki/Dining_philosophers_problem)
- [`pthread_create(3)`](https://man7.org/linux/man-pages/man3/pthread_create.3.html), [`pthread_mutex_lock(3p)`](https://man7.org/linux/man-pages/man3/pthread_mutex_lock.3p.html) and [`gettimeofday(2)`](https://man7.org/linux/man-pages/man2/gettimeofday.2.html) manuals
- [Unix Threads in C — CodeVault (YouTube playlist)](https://www.youtube.com/playlist?list=PLfqABt5AS4FmuQf70psXrsMLEDQXNkLq2)
- [Deadlock — Coffman conditions](https://en.wikipedia.org/wiki/Deadlock_(computer_science))
- [Helgrind / DRD — Valgrind thread error detectors](https://valgrind.org/docs/manual/hg-manual.html)
