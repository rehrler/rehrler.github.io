# Romans helpful helpers

## Math

### Mapping uniform area to uniform area

$$ \left[a,b\right] \overset{f}{\to} \left[c,d\right]$$
$$ f(x) = \frac{d-c}{b-a}\cdot\left(x-a\right)+c $$

### Statistical Confidence Interval

$$ \bar{x} = \frac{1}{n} \sum_{i=1}^{n} x_i $$
$$ S = \sqrt{\frac{1}{n-1} \sum_{i=1}^{n}\left(x_i - \bar{x}\right)} $$
$$ \mathcal{I}_{100\%-\alpha\%} = \left[\bar{x}\pm t_{n-1:1-\frac{\alpha}{200}}\cdot\frac{s}{\sqrt{n}}\right] $$


### Nonlinear Regression: Gauss-Newton Method

#### Given
A set of measurements $f_i(x_i)$ for certain locations $x_i, i\in\{1,\dots,n\}$. A nonlinear function $f(x_i,\underline{\theta})$ which has unknow parameters $\underline{\theta}=\left[\theta_1, \dots, \theta_m\right]^\top$.

#### Approach
- Approximate $f$ with the 1st order derivative

$$ f(x)\approx f(x_i,\underline{\tilde{\theta}})+\frac{\delta f}{\delta\theta_1}\cdot(\theta_1-\tilde{\theta}_1) + \dots + \frac{\delta f}{\delta\theta_j}\cdot(\theta_j-\tilde{\theta}_j) $$

- In matrix form:

$$ \underbrace{F(x)}_{\text{measurements at }x_i} = \underbrace{F(x, \underline{\tilde{\theta}})}_{\text{possible measurements at }x_i\text{ with guessed parameter }\underline{\tilde{\theta}}} + \underbrace{A\cdot(\underline{\theta}-\underline{\tilde{\theta}})}_{\text{difference of true and guessed parameter}} $$

#### Algorithm
1. Initialize $\underline{\tilde{\theta}}_{(1)}$ .
2. Do regression using 
$$\underline{\tilde{\theta}}_{(1)} \to\underline{\beta}_{(1)}=\underline{\theta}-\underline{\tilde{\theta}}_{(1)}$$
3. Calculate new 
$$\underline{\tilde{\theta}}_{(2)}=\underline{\tilde{\theta}}_{(1)}-\underline{\beta}_{(1)}$$
4. Repeat until $\underline{\beta}_{(i)}$ is small enough.

### Coordinate Systems

- World coordinate system $\mathcal{W}$ as the reference system.
- Coordinate System $\mathcal{B}$ at $P_\mathcal{W}^\mathcal{B}$ and $\beta_\mathcal{W}$.
- Coordinate System $\mathcal{N}$ at $P_\mathcal{W}^\mathcal{N}=$ and $\eta_\mathcal{W}$.

$$ T_{\mathcal{WB}} = \left[\begin{array}{cc}R_{\mathcal{WB}}(\beta_\mathcal{W}) & \mathbf{P}^\mathcal{B}_\mathcal{W} \\	0 & 1	\end{array}\right] $$
$$ T_{\mathcal{WN}} = \left[\begin{array}{cc}R_{\mathcal{WN}}(\eta_\mathcal{W}) & \mathbf{P}^\mathcal{N}_\mathcal{W} \\	0 & 1	\end{array}\right] $$

$$ T_{\mathcal{BW}} = T_{\mathcal{WB}}^{-1} $$
$$ T_{\mathcal{NW}} = T_{\mathcal{WN}}^{-1} $$

$$ T_{\mathcal{BN}} = T_{\mathcal{BW}}\cdot T_{\mathcal{WN}}$$

## Python

#### Parallelization for single function

```python
import concurrent.futures


class ParallelLoader(object):

    def __init__(self):
        pass

    def parallel_execution(self, entries: list) -> tuple:

        exp_2_res, exp_3_res = [], []
				
        # change max workers
        with concurrent.futures.ThreadPoolExecutor(max_workers=4) as executor:
            for result in executor.map(self._execute_single_task, entries):
                exp_2_res.append(result[0])
                exp_3_res.append(result[1])

        # return result
        return exp_2_res, exp_3_res

    def _execute_single_task(self, single_entry: float) -> tuple:
        # do something with single_entry
        return single_entry**2, single_entry**3
```

The effect becomes significant the larger the entry and the more cpu cores are available. 

## ROS

### ROS C++

#### Pointers and References

##### Call by value
Values of transported variables are copied. The original variables stay unchanged.

```cpp
void swapv(int a, int b) {
  /**
  changes not visible outside of function
  */
	int temp = a;
  a = b;
  b = temp;
}

int main(int argc, char** argv) {
	int wallet1 = 300;
  int wallet2 = 350;
  swapv(wallet1, wallet2); // wallet1 = 300, wallet2 = 350
  return 0;
}
```

##### Call by reference: Pointers
Addresses are copied instead of values. With these, it's possible to access and change the original variables.

```cpp
void swapp(int *p, int *q) {
	/**
  changes visible outside of function
  */
  int temp = *p;
  int *p = *q;
  *q = temp;
}

int main(int argc, char** argv) {
	int wallet1 = 300;
  int wallet2 = 350;
  swapp(&wallet1, &wallet2); // wallet1 = 350, wallet2 = 300
  return 0;
}
```

##### Call by reference: References
Aliases are created and the original variables can be accessed inside the function.

```cpp
void swapr(int &a, int &b) {
	/**
  no copy of address is created
  */
  int temp = a;
  a = b;
  b = temp;
}

int main(int argc, char** argv) {
	int wallet1 = 300;
  int wallet2 = 350;
  swapr(wallet1, wallet2); // wallet1 = 350, wallet2 = 300;
  return 0;
}
```

#### Example Makefile

```makefile
# Makefile for the rational example.
# builds program 'rational ' from the source files main.cc, rational.h, and rational.cc

CC =g ++
CFLAGS = -c

all : rational
rational : main.o rational.o
					 $(CC) -o rational main.o rational.o

main.o: main.cc rational.h
		 		$(CC) $(CFLAGS) main.cc
         
rational.o: rational.cc rational.h
					  $(CC) $(CFLAGS) rational.cc
            
clean : rm -f *.o rational
```

#### Inline callback function using boost

```cpp
#include <ros/ros.h>
#include <sensor_msgs/Joy.h>

int main(int argc, char **argv) {
	ros::init(argc, argv, "example_node");
  ros::NodeHandle n;
  
  # define subscriber callback with type sensor_msgs::Joy
  boost::function<void(const sensor_msgs::Joy::ConstPtr &)> joy_callback;
  
  joy_callback = [&](const sensor_msgs::Joy::ConstPtr &joy_msg) {
  	# do something with joy_msg
    int button_value = joy_msg->buttons[1];
	}
  
  # assign subscriber
  ros::Subscriber joy_sub = n.subscribe("/joy", 1, joy_callback);
}
```

### ROS Python

#### Class callback function

```python
import rospy
from sensor_msgs.msg import Joy


class ExampleSub(object):
	def __init__(self):
  	pass
    
  def start(self):
  	rospy.init_node("example_node")
    self._joy_sub = rospy.Subscriber("/joy", Joy, self._joy_callback)
    rospy.spin()
	
  def _joy_callback(self, joy_msg):
  	# do something with joy_msg
    button = joy_msg.data.buttons[1]
    
 
if __name__ == '__main__':
	try:
  	ex_sub = ExampleSub()
    ex_sub.start()
	except rospy.ROSInterruptException as e:
  	print(e)
```

### Other helpful stuff

#### Example docker file for single ros node

```docker
FROM ros:melodic

RUN apt-get update && apt-get install -y \
    python-catkin-tools \ # may be adapted to install other depending packages
    && rm -rf /var/lib/apt/lists/*

ENV ROS_WS /opt/ros_ws
RUN mkdir -p $ROS_WS/src/this_package/ # may be renamed
COPY . $ROS_WS/src/this_package/

WORKDIR $ROS_WS
RUN catkin config --extend /opt/ros/$ROS_DISTRO \
    && catkin init \
    && rosdep install --from-paths src --ignore-src -r -y \
    && catkin build

RUN sed --in-place --expression \
    '$isource "$ROS_WS/devel/setup.bash"' \
    /ros_entrypoint.sh
```

Placed in the package, one can build the container and execute it to use the local roscore
```bash
cd this_package/
docker build -t this_package_docker_container .
docker run -it --net=host this_package_docker_container roslaunch this_package this_package.launch
```

#### ROS network across multiple machines

source: [ROS Wiki](http://wiki.ros.org/ROS/Tutorials/MultipleMachines)

| machine | shell sign | example ip | example hostname |
| --- | --- | --- | --- |
| A | `$` | `10.0.0.1` | `machine_a` |
| B | `%` | `10.0.0.2` | `machine_b` |

- Check if the machines can ping each other
```bash
$ ping 10.0.0.2       # or ping machine_b
% ping 10.0.0.1       # or ping machine_a
```

- Add two shell variables in `machine_a` and start the roscore. *ATTENTION*: If you open a new shell, you have to enter
  the variables again. Therefore it is recommended to add the commands to the `.bashrc`.
```bash
$ export ROS_MASTER_URI=http://10.0.0.1:11311
$ export ROS_IP=10.0.0.1 # here the own ip of the server machine_a
$ roscore
```

- Add the same shell variables in `machine_b` but with the `ROS_IP` as the ip of `machine_b`. As above, it's recommended
  to add these variables into the `.bashrc`.
```bash
% export ROS_MASTER_URI=http://10.0.0.1:11311
% export ROS_IP=10.0.0.2 # here the ip of the client machine_b
% rostopic list
/rosout
/rosout_agg
```

- Check with an example topic, if the communication is definitely bidirectional.


## Software Engineering

### Example yaml file for CI/CD on gitlab
```yaml
image: ros:melodic-ros-core

cache:
  paths:
    - ccache/

before_script:
  - apt update >/dev/null && apt install -y git >/dev/null
  - git clone https://gitlab.com/VictorLamoine/ros_gitlab_ci.git >/dev/null
  - source ros_gitlab_ci/gitlab-ci.bash >/dev/null
  # add additional required packages here

catkin_make:
  stage: build
  script:
    - catkin_make

catkin tools:
  stage: build
  script:
    - catkin build --summarize --no-status --force-color
```

### Screen

The following commands are useful for [screen](https://linuxize.com/post/how-to-use-linux-screen/):
	
- create a virtual terminal sessino: 
```bash
screen -S <name_of_session>
```
- leave virtual terminal session with `CTRL+a, CTRL+d` without closing the execution task.
- list all sessions and virtual terminals: 
```bash
screen -ls
```
- reconnect to previous running session: 
```bash
screen -r <name_of_session>
```
- closing the current session: 
```bash
exit
```

### File encryption & decryption

#### Encryption
- Files: `gpg -c filename`
- Directories: `gpg-zip -c -o output_file.gpg dirname`

#### Decryption
- Files: `gpg filename.gpg`
- Directories: `gpg-zip -d filename.gpg`

### Preferred tools

#### IDE

- CLion for c++ and python development. [ROS guide](https://www.jetbrains.com/help/clion/ros-setup-tutorial.html)

- PyCharm Professional for pure python development

- Rider for c#

Registration for student/academic access with hslu mail: [jetbrains.com](https://www.jetbrains.com/shop/eform/students)

- visual studio code for ssh development

- vim with `.vimrc` and install plugins in vim using `:PlugInstall`.

```
set nocompatible
filetype on
filetype plugin on
filetype indent on
syntax on
set number
set cursorline
set cursorcolumn
set shiftwidth=4
set tabstop=4
set expandtab
set nobackup
set scrolloff=10
set nowrap
set incsearch
set ignorecase
set smartcase
set showcmd
set showmode
set showmatch
set hlsearch
set history=1000
set wildmenu

colo molokai

set wildmode=list:longest

set wildignore=*.docx,*.jpg,*.png,*.gif,*.pdf,*.pyc,*.exe,*.flv,*.img,*.xlsx

" PLUGINS ---------------------------------------------------------------- {{{

call plug#begin('~/.vim/plugged')

  Plug 'dense-analysis/ale'
  Plug 'preservim/nerdtree'
  Plug 'taketwo/vim-ros'
  Plug 'Valloric/YouCompleteMe', { 'commit':'d98f896' }

call plug#end()

" }}}

```

#### Linux

- KDE desktop environment
- terminator as terminal application
- password manager bitwarden
- ublock for firefox
- luks disk encryption

##### Commands

- `find <path> <command>`, ex.: `find ./ -name "*.cc"`
- `find <path> <command> -exec <command> \`, ex.: `find . -name core -exec rm {}\`
- `kill -9 <pid>`
- `alias <aliasname> <command>`

##### Video manipulation

- cuts input_file video from xx secs until yy seconds are reached with a speedup of 1/0.z, scaling of a and removed audio.
```bash 
ffmpeg -ss 00:00:xx -i <input_file> -t 00:00:yy -async 1 -an -vf "setpts=0.z*PTS,scale=a:-1" <output_file> 
```
- creates gif from images of type *.png and loops
```bash
convert -delay 20 -loop 0 *.png myimage.gif
```

#### Resources

- [git guide](https://gitlab.uzh.ch/zi-it-training/git/-/blob/master/git.pdf)
- [bash guide](https://thealternative.ch/guides/bash.php)
- [clion ros docker development](https://www.allaban.me/posts/2020/08/ros2-setup-ide-docker/)


### LaTeX

#### Vector based graphics

1. Create in inkscape graphic and use latex math mode
2. Export as pdf_tex
3. Include file in .tex
```latex
\begin{figure}[h!]
		\centering
		\def\svgwidth{0.7\columnwidth}
		\input{sketch.pdf_tex}
\end{figure}
```

## Linux at HSLU

### Network setup

#### LAN

Use 802.1x Security:
1. Authentication: Protected EAP (PEAP)
2. CA: certificate: [quovadis_rca2_der.crt](https://www.hslu.ch/-/media/campus/common/files/dokumente/h/helpdesk/anleitungen/netzwerk/802-1x/8021x-linux.pdf?la=en)
3. PEAP version: Automatic
4. Inner authentification: MSCHAPv2
5. Username: campus\username
6. Password: ...

#### Labor

| name | value | 
| --- | --- |
| address | `10.180.30.x` |
| netmask | `255.255.255.0` |
| gateway | `0.0.0.0` |

#### WLAN

Use 802.1x Security:
1. Security: WPA/WPA2 Enterprise
2. Authentication: Protected EAP (PEAP)
3. PEAP version: Automatic
4. Inner authentification: MSCHAPv2
5. Username: campus\username
6. Password: ...

### Network drives

#### Method 1: file explorer in linux

- Go to *Other Locations*, then *Connect to Server*: `smb://fs01e.eee.intern/data$/30\ CCMS/` for CC MS network drive f.e. 
- Use credentials of eee Portal with Domain `eee`. Example:
```
Login: eee\eee-username
PW: eee-passwort
```

#### Method 2: mount on specific folder using terminal

- Create target folder
```bash
mkdir -p ~/mounts/x_ccms
```

- Mount the network drive on this folder:
```bash
sudo mount -t cifs -o "domain=eee,username=eee-username,password=eee-passwort" //fs01e.eee.intern/data$/30\ CCMS/ /home/username/mounts/x_ccms     
```

> With this method, read and write rights are only for `root`. Therefore, use `sudo` to copy, delete, move or edit files.
{.is-warning}


- Unmount the network drive in this folder:
```bash
sudo umount /home/username/mounts/x_ccms
```

