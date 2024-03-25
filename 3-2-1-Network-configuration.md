# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## Network configuration

* Create additional network interface

* Configure static `10.10.x.1/24` network on it 
  * Use `nmtui` to create `enp0s8` interface
  * Assign static IP address `10.10.x.1`
    * x=1-9
    * `10.10.x.111` address will be teacher's IP in each student's subnet
    

This example is based on the environment like follows.
```bash
--------+---------------------+----------------------+------------
        | [enp0s8]            | [enp0s8]             | [enp0s8]
        | 10.10.0.1           | 10.10.1.1            | 10.10.2.1
        | 10.10.1.111         |                      |
        | 10.10.2.111         |                      |
        | 10.10.x.111         |                      |
+-------+--------+   +--------+---------+   +--------+---------+
|     lt00.am    |   |      lt01.am     |   |     lt02.am      |
|     Teacher    |   |    Student 1     |   |    Student 2     |
+----------------+   +------------------+   +------------------+

```