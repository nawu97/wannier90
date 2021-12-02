# qe中关于wannier90的使用(利用wannier构造紧束缚模型)
## 步骤：
1）用pw.x运行自洽scf和非自洽nscf


2) 准备.win文件，运行wannier90.x -pp 生成seedname.nnkp文件


3) 运行pw2wannier90.x读入pw输出文件，seedname.pw2wan文件（（接口文件seedname.win和seedname.nnkp, 生成seedname.mmn, seedname.amn和seedname.eig）


4) 运行wannier90.x得到能带

命令汇总：
```
mpirun -n 4 pw.x < silicon.scf > silicon.scf.out
mpirun -n 4 pw.x < silicon.nscf > silicon.nscf.out
wannier90.x -pp silicon
mpirun -n 4 pw2wannier90.x < silicon.pw2wan > silicon.pw2wan.out
wannier90.x silicon 
mpirun -n 4 pw.x < silicon.band > silicon.band.out
mpiexec -n 4 bands.x < band.in
```
## 示例（以wannier90 example中example11为例）：
### 1）用pw.x运行自洽scf和非自洽nscf
#### 自洽：
```
Silicon
 &control
    calculation     =  'scf'
    restart_mode    =  'from_scratch'
    prefix          =  'si'               #si.save 中的si
    tprnfor         =  .true.
    pseudo_dir      =  '../../pseudo/'    #指定赝势
    outdir          =  './'                # si.save 的位置
    iprint          =   2
/
 &system
    ibrav           =   2                   #晶格类型
    celldm(1)       =  10.2                 #晶格常数 单位bohr
    nat             =   2                   #原子总数
    ntyp            =   1                   #原子类型数
    ecutwfc         =  25.0                 #截断能，单位Ry
/
 &electrons
    conv_thr        =   1.0d-12             #电子收敛精度
    diagonalization =  'cg'                 #电子优化算法
/
ATOMIC_SPECIES
 Si  28  Si.pbe-n-van.UPF                   #指定赝势文件
ATOMIC_POSITIONS {crystal}                  # crystall 为分数坐标
Si  -0.25   0.75   -0.25                    # Si的位置
Si   0.00   0.00    0.00      
K_POINTS {automatic}                        # 自动生成k点
20 20 20 0 0 0
```
#### 非自洽：
非自洽生成k点的方法：(kmesh写出在kpoint下。kmesh.pl脚本来自wannier90软件的uitility文件夹下)
```
kmesh.pl 8 8 8  > kpoint
```

```
Silicon
 &control
    calculation        =  'nscf'         # 修改这里
    prefix             =  'si'
    pseudo_dir         =  '../../pseudo/'
    outdir             =  './'
    iprint             =   2
    verbosity          =  'high'         #输出本征值     
/
 &system
    ibrav              =    2  
    celldm(1)          =   10.2
    nat                =    2
    ntyp               =    1
    ecutwfc            =   25.0
    nbnd               =   12              #设定能带数
/
 &electrons
    conv_thr           =   1.0d-12
    diagonalization    =  'cg'
    diago_full_acc     =  .true.           #对角化
/
ATOMIC_SPECIES
 Si 28 Si.pbe-n-van.UPF
ATOMIC_POSITIONS {crystal}
Si  -0.25    0.75   -0.25
Si   0.00    0.00    0.00
K_POINTS {crystal}
512
  0.00000000  0.00000000  0.00000000  1.953125e-03
  0.00000000  0.00000000  0.12500000  1.953125e-03
  ...
  ...
  0.87500000  0.87500000  0.75000000  1.953125e-03
  0.87500000  0.87500000  0.87500000  1.953125e-03
  ```
  ### 2）准备.win文件，运行wannier90.x -pp 生成seedname.nnkp文件
  #### a)wannier90.x -pp silicon 生成生成silicon.nnkp文件。
  #### b)win文件的k点同样可以用mesh.pl生成，命令为kmesh.pl 8 8 8 wan > kpoint-wan 
  ```
  num_bands        =   12

num_iter         =  100
dis_num_iter     =  100

iprint           =    2
num_dump_cycles  =   10
num_print_cycles =   10

length_unit      =  bohr


begin projections
!! !! Bond-centred s-orbitals
f=-0.125,-0.125, 0.375:s
f= 0.375,-0.125,-0.125:s
f=-0.125, 0.375,-0.125:s
f=-0.125,-0.125,-0.125:s
!! !! Atom-centred sp3-orbitals
Si:sp3
end projections

!! (1) Valence bands
num_wann        =   4
select_projections 1 2 3 4
dis_froz_max    =   6.5
dis_win_max     =   6.5

!! !! (2) Valence + conduction bands
!! num_wann        =   8
!! select_projections 5-12
!! dis_froz_max    =   6.5
!! dis_win_max     =  17.0


begin unit_cell_cart
 bohr
-5.10   0.00   5.10
 0.00   5.10   5.10
-5.10   5.10   0.00
end unit_cell_cart


begin atoms_frac
Si -0.25  0.75  -0.25
Si  0.00  0.00   0.00
end atoms_frac


!! !! To plot the WF interpolated bandstructure
!! restart         =  plot
!! bands_plot      =  true
!! begin kpoint_path
!! L 0.50000  0.50000 0.5000 G 0.00000  0.00000 0.0000
!! G 0.00000  0.00000 0.0000 X 0.50000  0.00000 0.5000
!! X 0.50000 -0.50000 0.0000 K 0.37500 -0.37500 0.0000
!! K 0.37500 -0.37500 0.0000 G 0.00000  0.00000 0.0000
!! end kpoint_path


!! !! To plot the WFs
!! restart                =  plot
!! wannier_plot           =  true
!! wannier_plot_supercell =  3
!! wannier_plot_list      =  1,5


mp_grid  =  4 4 4

begin kpoints
  0.00000000  0.00000000  0.00000000
  0.00000000  0.00000000  0.25000000
  0.00000000  0.00000000  0.50000000
  0.00000000  0.00000000  0.75000000
  0.00000000  0.25000000  0.00000000
  0.00000000  0.25000000  0.25000000
  0.00000000  0.25000000  0.50000000
  0.00000000  0.25000000  0.75000000
  0.00000000  0.50000000  0.00000000
  0.00000000  0.50000000  0.25000000
  0.00000000  0.50000000  0.50000000
  0.00000000  0.50000000  0.75000000
  0.00000000  0.75000000  0.00000000
  0.00000000  0.75000000  0.25000000
  0.00000000  0.75000000  0.50000000
  0.00000000  0.75000000  0.75000000
  0.25000000  0.00000000  0.00000000
  0.25000000  0.00000000  0.25000000
  0.25000000  0.00000000  0.50000000
  0.25000000  0.00000000  0.75000000
  0.25000000  0.25000000  0.00000000
  0.25000000  0.25000000  0.25000000
  0.25000000  0.25000000  0.50000000
  0.25000000  0.25000000  0.75000000
  0.25000000  0.50000000  0.00000000
  0.25000000  0.50000000  0.25000000
  0.25000000  0.50000000  0.50000000
  0.25000000  0.50000000  0.75000000
  0.25000000  0.75000000  0.00000000
  0.25000000  0.75000000  0.25000000
  0.25000000  0.75000000  0.50000000
  0.25000000  0.75000000  0.75000000
  0.50000000  0.00000000  0.00000000
  0.50000000  0.00000000  0.25000000
  0.50000000  0.00000000  0.50000000
  0.50000000  0.00000000  0.75000000
  0.50000000  0.25000000  0.00000000
  0.50000000  0.25000000  0.25000000
  0.50000000  0.25000000  0.50000000
  0.50000000  0.25000000  0.75000000
  0.50000000  0.50000000  0.00000000
  0.50000000  0.50000000  0.25000000
  0.50000000  0.50000000  0.50000000
  0.50000000  0.50000000  0.75000000
  0.50000000  0.75000000  0.00000000
  0.50000000  0.75000000  0.25000000
  0.50000000  0.75000000  0.50000000
  0.50000000  0.75000000  0.75000000
  0.75000000  0.00000000  0.00000000
  0.75000000  0.00000000  0.25000000
  0.75000000  0.00000000  0.50000000
  0.75000000  0.00000000  0.75000000
  0.75000000  0.25000000  0.00000000
  0.75000000  0.25000000  0.25000000
  0.75000000  0.25000000  0.50000000
  0.75000000  0.25000000  0.75000000
  0.75000000  0.50000000  0.00000000
  0.75000000  0.50000000  0.25000000
  0.75000000  0.50000000  0.50000000
  0.75000000  0.50000000  0.75000000
  0.75000000  0.75000000  0.00000000
  0.75000000  0.75000000  0.25000000
  0.75000000  0.75000000  0.50000000
  0.75000000  0.75000000  0.75000000
end kpoints
  ```
  ### 3)运行pw2wannier90.x读入pw输出文件，seedname.pw2wan文件
  ```
  &inputpp
  outdir     =  './'
  prefix     =  'si'        # 对应si.save  
  seedname   =  'silicon'   # 对应silicon.win
  write_amn  =  .true.
  write_mmn  =  .true.
/
  ```
  ### 4)运行wannier90.x得到能带
  .win文件中能带路径为：
```
bands_plot      =  true
begin kpoint_path
L 0.50000  0.50000 0.5000 G 0.00000  0.00000 0.0000
G 0.00000  0.00000 0.0000 X 0.50000 -0.50000 0.0000
X 0.50000 -0.50000 0.0000 G 0.00000  0.00000 0.0000
end kpoint_path
```
运行

wannier90.x silicon &
能带文件为silicon_band.dat
