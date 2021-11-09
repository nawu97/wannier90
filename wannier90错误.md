# wannier90 常见的错误汇总
## 1.
```
Exiting......
kmesh_get_bvector: Not enough bvectors found
```
原因：临近的b矢量太少了，因为这时候kmesh的收敛标准太高了
改正：
```
kmesh_tol = 0.0001
```
## 2.
```
task #         0
     from pw2wannier90 : error #         5
     Direct lattice mismatch

```
原因：win中的
```
begin unit_cell_cart
        3.8564786911         0.0000000000         0.0000000000
       -1.9340042258         3.3364748978         0.0000000000
        0.0000000000         0.0000000000        17.4066505432
end unit_cell_cart
```
与scf/nscf的结构应该一致

## 3.
```
task #         0
     from pw2wannier90 : error #         5
     Reciprocal lattice mismatch

```
仔细输进去，一位也不要有误差！
