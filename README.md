[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/JdNTJJYf)
# assignment-7
module load nvhpc

module load cuda

module load craype-accel-nvidia80

nvc++ -mp=gpu -gpu=cc80 -Ofast laplace2d.cpp -o laplace -Minfo=accel,mp

srun -p gpu --gres=gpu:1 --ntasks=1 --time=00:05:00 --mem=40G ./laplace

# notes:
module load nvhpc
nvc++ laplace2d.cpp -mp -Ofast -o laplace_cpu

srun -p cpu --cpus-per-task=4 --mem-per-cpu=2000 --time=00:05:00 ./laplace_cpu

if ran ffor 1000 iter -- it only takes 4 times more not 10 times -- why? cuz most of this time is taken up by the copying of the data between the cpu and gpu, and that doesnt scale with the iteration change

copy+compute(100) = 0.7
copy+compute(1000) = 4.1
(-)
compute(900) = 3.4
compute = 0.00377
copy = 0.7 - 100*0.00377  (0.37)
copy = 0.3235 -- copying takes almost as much time as the computation itself; 
but if we do a 1000 iteration, the compute will be ~3.78 seconds and the copy will still take the same time

if only done by cpu (100->1000) ~17 sec
--> if you load a shit ton of computation to the gpu then in the grand sceme of things this copying time will be worth it

with adding collapse(2)
(100) - copy+compute=0.4 (for me 384ms) -- copy is the same compute is 0.00077  ??
(1000) - (0.7 +0.323) ~ 1 (for me it was 838 ms ?)  --- 17times faster then cpu  


omit all data clauses:
nvc++ laplace2d.cpp -mp=gpu -Ofast -Minfo=accel,mp -o laplace2d_acc -gpu=cc80,managed
