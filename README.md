# Optimal Actuator Location for Railway Tracks
Actuator location and design are important design variables  in controller synthesis for distributed parameter systems. Finding the best actuator location to control a distributed parameter system can significantly reduce the cost of the control and improve  its effectiveness. From a theoretical point of view, the existence of an optimal actuator location has been discussed in the literature for linear partial differential equations (PDE's). It was proven that an optimal actuator location exists for linear-quadratic control.  Conditions under which using approximations in optimization yield the optimal location are also established.
 Similar results have been obtained for $H_2$ and $H_\infty$ controller design objectives with linear models. There are also results on the related problem of optimal sensor location for linear PDE's.



## Raiway Track Model
Railway tracks are rested on ballast which are known for exhibiting nonlinear viscoelastic behavior. If a track beam is made of a Kelvin-Voigt material, then the railway track model will be a semi-linear partial differential equation on $\xi\in [0,\ell]$ as follows:

\begin{flalign}\notag
\begin{cases}
\begin{aligned}
&\rho a \frac{\partial^2 w}{\partial t^2}+\frac{\partial}{\partial \xi^2}(EI\frac{\partial^2 w}{\partial \xi^2}+C_d\frac{\partial ^3 w}{\partial \xi^2 \partial t})+\mu \frac{\partial w}{\partial t}+kw+\alpha w^3=b(\xi;r)u(t), &\xi&\in(0,\ell),\\[2mm]
&w(\xi,0)=w_0(\xi), \quad \frac{\partial w}{\partial t}(\xi,0)=v_0(\xi), &\xi&\in(0,\ell),\\[2mm]
&w(0,t)=w(\ell,t)=0,&t&\ge 0,\\[2mm]
&EI\frac{\partial ^2w}{\partial \xi^2}(0,t)+C_d\frac{\partial ^3w}{\partial \xi^2\partial t}(0,t)=EI\frac{\partial^2 w}{\partial \xi^2}(\ell,t)+C_d\frac{\partial ^3w}{\partial \xi^2\partial t}(\ell,t)=0, &t&\ge 0.
\end{aligned}
\end{cases}
\end{flalign}

where the positive constants $E$, $I$, $\rho$, $a$, and $\ell$ are the modulus of elasticity, second moment of inertia, density of the beam, cross-sectional area, and length of the beam, respectively. The linear and nonlinear parts of the foundation elasticity correspond to the coefficients $k$ and $\alpha$, respectively. The constant $\mu\ge 0$ is the viscous damping coefficient of the foundation, and $C_d\ge 0$ is the coefficient of Kelvin-Voigt damping in the beam.
The track deflection is controlled by an external force  $u(t)$;  $u(t)$  will  be
assumed to be a scalar input in order to simplify the exposition. The shape influence function $b(\xi;r)$ is a continuous function over $[0,\ell]$ parametrized by the parameter $r$ that describes its dependence on actuator location. For example, as shown in the next figure, the control force is typically localized at some point $r$ and $b(\xi;r)$ models the distribution of the force $u(t)$ along the beam. The
function $b(\xi;  r) $  is assumed continuously differentiable with respect to $r$ over $\mathbb{R}$; a suitable function for the case of actuator location is illustrated in the figure.

<p align="center">
<img src="figs/Beam-schematic-2.JPG" width="400" />
</p>






## Galerkin Solver
This repository contains codes for solve nonlinear PDE models

```Matlab
%% Generator
% This script generates the nonlinear functions appeared in the truncated
% nonlinear system for any order of approximation
tic
clc
clear all
N=20  %Specify the maximum order of approximation

syms alpha l x r delta EI rhoa k l par
assume(alpha>0 & l>0 & r>0 & EI>0 & rhoa>0 & k>0 & l>0 & delta>0)
assumeAlso(2*delta<l & delta<r & r+delta<l)
par=[EI,rhoa,k,l,alpha,delta];

b=piecewise(r-delta<x<r+delta,(x-r+delta)^2*(x-r-delta)^2/(16*delta^5)*15,0);
br=diff(b,r);
%%
z=[];
cc=[];
W=0;

for n=1:N
   z=[z;sym(sprintf('z%d',n))];
end
for n=1:N/2
   cc=[cc,sym(sprintf('c%d',n))];
   W=W+z(2*n-1)*cc(n)*sin(n*pi*x/l)/(n^2*pi^2)...
      -z(2*n)*cc(n)*sin(n*pi*x/l)/(n^2*pi^2);
   for i=1:n
      F(2*i-1,1)=-alpha*cc(i)*simplify(int(W^3*sin(i*pi*x/l),x,0,l));
      F(2*i,1)=F(2*i-1,1);
      
      B(2*i-1,1)=cc(i)*simplify(int(b*sin(i*pi*x/l),x,0,l));
      B(2*i,1)=B(2*i-1,1);
      
      dB(1,2*i-1)=cc(i)*simplify(int(br*sin(i*pi*x/l),x,0,l));
      dB(1,2*i)=dB(1,2*i-1);
   end
   
   for i=1:n
      for j=1:n
         dF(2*i-1,2*j-1)=-3*alpha*cc(i)*cc(j)*simplify(int(W^2*sin(i*pi*x/l)*sin(j*pi*x/l),x,0,l));
         dF(2*i,2*j-1)=dF(2*i-1,2*j-1);
         dF(2*i-1,2*j)=dF(2*i-1,2*j-1);
         dF(2*i,2*j)=dF(2*i-1,2*j-1);
      end
   end
   F=subs(F,cc,c(1:n,par));
   dF=subs(dF,cc,c(1:n,par));
   B=subs(B,cc,c(1:n,par));
   dB=subs(dB,cc,c(1:n,par));
   
   matlabFunction(F,'File',sprintf('F%d',2*n),'Vars',{z,par});
   matlabFunction(dF,'File',sprintf('dF%d',2*n),'Sparse',true,'Vars',{z,par});
   matlabFunction(B,'File',sprintf('B%d',2*n),'Vars',{r,par});
   matlabFunction(dB,'File',sprintf('dB%d',2*n),'Vars',{r,par});
   n
end
toc
```
