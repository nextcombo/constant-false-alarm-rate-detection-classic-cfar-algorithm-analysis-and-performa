# [Constant False Alarm Rate Detection]Classic CFAR Algorithm Analysis and Performance Chart Display [Attached MATLAB Code]

The premise is that the receiver output contains noise (including atmospheric noise, human-made noise, internal noise, and clutter, etc.), and the signal generally superimposes on the noise. This requires the use of detection technology to determine whether there is a target signal under the condition of added noise in the receiver output or signal.
Error probability Any form of decision inevitably involves two error probabilities: detection probability and false alarm probability. When there is a target echo signal at the receiver output and the probability of making a decision as having a target is Pd, and the probability of making a decision as not having a target is 1 - Pd. When there is only noise at the receiver output and the probability of making a decision as having a target is Pfa. Since noise is a random variable, its characteristic can be represented by a probability density function, so the signal plus noise is also a random variable [1].
Specific process The constant false alarm detector first processes the input noise to determine a threshold, comparing this threshold with the input signal. If the input signal exceeds this threshold, it is judged as having a target, otherwise, it is judged as not having a target.
Method To keep the false alarm probability constant, the threshold is adjusted rapidly according to the change in the input noise. The noise processing method varies with the different distribution of noise, hence the constant false alarm detection technology includes constant false alarm processing technology and target detection technology. The constant false alarm processing technology includes fast gate, slow gate processing technology, target detection including likelihood ratio detection, binary detection, sequential detection, and nonparametric detection, etc.
Classical CFA algorithms include:
① Mean class CFA algorithms
The core idea is to estimate the background power by taking the average of the sampling data within the reference window.
CA-CFAR, GO-CFAR, SO-CFAR algorithms are the most classic mean class CFA algorithms, followed by their log-CFAR algorithm but the principle is the same.
Figure 1 Mean class CFA detector principle diagram
② Statistically ordered CFA
The core idea: By sorting the data within the reference window from small to large and selecting the Kth value as clutter background noise.
Figure 2 OS-CFAR detector principle diagram
③ Adaptive CFA
Different decision methods are adopted for different types of clutter.
Figure 3: HCE-CFAR detector principle diagram
Figure 4: VI-CFAR detector principle diagram
④ Clutter chart CFAR
The classic spatial clutter chart CFAR algorithm assumes that the squared rates on each range and azimuth are independent and identically distributed, and it fits the clutter background noise by forgetting iteration on the time dimension.
Figure 5: Clutter chart CFAR detector principle diagram
III. Current Development of CFAR
Mean-based - statistical ordered - adaptive - clutter chart - transform domain - compression sense - neural network
IV. Algorithm Implementation
cfar_ac.m: Unit average constant false alarm rate algorithm implementation
Advantages: The least loss rate among algorithms;
Disadvantages: Multi-target masking, poor performance on clutter edges.
function [ index, XT ] = cfar_ac( xc, N, pro_N, PAD)\n% Assuming the echo follows a Gaussian distribution.\n%\nalpha=N.*(PAD.^(-1./N)-1);\nindex=1+N/2+pro_N/2:length(xc)-N/2-pro_N/2;\nXT=zeros(1,length(index));\n\nfor i=index\n    cell_left=xc(1,i-N/2-pro_N/2:i-pro_N/2-1);\n    cell_right=xc(1,i+pro_N/2+1:i+N/2+pro_N/2);\n    Z=(sum(cell_left)+sum(cell_right))./N;\n\n    XT(1,i-N/2-pro_N/2)=Z.*alpha;\nend\n\nend\n  
%% Program initialization\nclc;clear all;close all;\n\n%% Uniform background noise (single target & multi-target)\n% shape=[200];\n% variance=200;\n% noise_db=20;\n% noise_p=10.^(noise_db./10);\n% show_out=0;\n% [ xc ] = env_uniform(variance,  shape, noise_db,show_out);\n\n% Single target\n% SNR1=15;    signal1_p=10.^(SNR1./10).*noise_p;\n% xc(1,90)=signal1_p;\n%\n% Multi-target\n% SNR1=20;signal1_p=10.^(SNR1./10).*noise_p;\n% xc(1,90)=signal1_p;\n% SNR2=15;signal2_p=10.^(SNR2./10).*noise_p;\n% xc(1,102)=signal2_p;\n\n\n%% Background noise of clutter edges (single target & multi-target)\nshape=[100,200];\nvariance=200;\nnoise_db=[20,30];\nnoise_p=10.^(noise_db./10);\nshow_out=0;\n% [ xc ] = env_edge(variance,  shape, noise_db, show_out);\n\n% Multi-target\nSNR1=15;signal1_p=10.^(SNR1./10).*noise_p(1,end);\nSNR2=12;signal2_p=10.^(SNR2./10).*noise_p(1,end);\nSNR3=8;signal3_p=10.^(SNR3./10).*noise_p(1,end);\nSNR4=5;signal4_p=10.^(SNR4./10).*noise_p(1,end);\n\n% loc1=randi([43,44],1,1);\n% xc(1,loc1)=signal1_p;\n% loc2=randi([46,48],1,1);\n% xc(1,loc2)=signal3_p;\n% loc3=randi([50,53],1,1);\n% xc(1,loc3)=signal2_p;\n% loc4=randi([55,58],1,1);\n% xc(1,loc4)=signal1_p;\n% loc5=randi([90,93],1,1);    % close to the clutter area, but still in\n% xc(1,loc5)=signal1_p;\n% xc(1,loc5)=signal2_p;     % This situation of clutter edge with target, it needs to be optimized.\n% xc(1,loc5)=signal3_p;\n\n% loc6=randi([102,108],1,1);\n% xc(1,loc6)=signal3_p;\n\n%% Iterative Algorithm Implementation (Scatterplot)\nN=36;\nu=0.001;\nPAD=10^(-4);\nalpha=N.*(PAD.^(-1./N)-1);\nxtp=zeros(1,shape(end));\nd0=0;\nd1=[0];\nfor i=1:1:4000\n    [ xc ] = env_edge(variance,  shape, noise_db,show_out);\n\n    loc1=randi([43,44],1,1);\n    xc(1,loc1)=signal1_p;\n    loc2=randi([46,48],1,1);\n    xc(1,loc2)=signal3_p;\n    loc3=randi([50,53],1,1);\n    xc(1,loc3)=signal2_p;\n    loc4=randi([55,58],1,1);\n    xc(1,loc4)=signal1_p;\n\n    loc6=randi([102,108],1,1);\n    xc(1,loc6)=signal3_p;\n\n    xtp=xtp.*(1-u)+xc.*u;\n    d0=d0.*(1-u)+u;\n    d1=cat(2,d1,d0);\n\n    if mod(i,100)==0\n        %% Map display\n        figure;\n        plot(10.*log(abs(xc))./log(10)),hold on;\n        plot(10.*log(abs(xtp.*alpha./d0))./log(10)),hold on;\n    end\n\nend\n\n  
function [ xc ] = env_uniform(variance,  shape, power_db, show_out)\n%UNIFORM_EN This displays a summary of this function\n% Here is the detailed description\nif (nargin==3)\n    show_out=0;\nend\n\nc=10^(power_db/10);     % Here is the amplitude - power\nxc=c + random('Normal',0,variance,1,shape);\nif show_out==1\n    figure;plot(10.*log(abs(xc))./log(10));\nend\n\nend\n  
function [ xc ] = env_edge(variance,  shape, power_db, show_out)\n%UNIFORM_EN This displays a summary of this function\n% Here is the detailed description\nif (nargin==3)\n    show_out=0;\nend\n\nc=10.^(power_db./10);               % Here is the amplitude - power\nxc=random('Normal',0,variance,1,shape(1,end));\nxc(1,1:end)=xc(1,1:end)+c(1,1);\nindex=1;\nfor i=1:length(power_db)\n    xc(1,index:shape(1,i))=xc(1,index:shape(1,i)).*c(1,i)./c(1,1);\n    index=shape(1,i)+1;\nend\n\n\nif show_out==1\n    figure;plot(20.*log(abs(xc))./log(10));\nend\n\nend\n  
cfar_go.m: Maximum Selection of Constant False Alarm Rate Algorithm Implementation
Advantages: Decrease in false alarm probability in clutter edge regions
Disadvantages: Multi-target masking
function [ index, XT ] = cfar_go( xc, N, pro_N, PAD)\n% Assuming the echo follows a Gaussian distribution.\n%   Alpha assignment has some issues, a more complex high-degree function\nalpha=N.*(PAD.^(-1./N)-1);\n\nindex=1+N/2+pro_N/2:length(xc)-N/2-pro_N/2;\nXT=zeros(1,length(index));\n\nfor i=index\n    cell_left=xc(1,i-N/2-pro_N/2:i-pro_N/2-1);\n    cell_right=xc(1,i+pro_N/2+1:i+N/2+pro_N/2);\n    Z=max([mean(cell_left),mean(cell_right)]);\n\n    XT(1,i-N/2-pro_N/2)=Z.*alpha;\nend\n\nend\ncfar_so.m Implementation of Minimum Selective False Alarm Algorithm  
Advantages: Improved performance for multi-target effects; Disadvantages: Increased false alarm probability in cluttered edge areas.
function [ index, XT ] = cfar_so( xc, N, pro_N, PAD)\n% Assume that the echo follows a Gaussian distribution.\n%   Alpha assignment has some issues, a more complex high-order function\nalpha=N.*(PAD.^(-1./N)-1);\n\nindex=1+N/2+pro_N/2:length(xc)-N/2-pro_N/2;\nXT=zeros(1,length(index));\n\nfor i=index\n    cell_left=xc(1,i-N/2-pro_N/2:i-pro_N/2-1);\n    cell_right=xc(1,i+pro_N/2+1:i+N/2+pro_N/2);\n    Z=min([mean(cell_left),mean(cell_right)]);\n\n    XT(1,i-N/2-pro_N/2)=Z.*alpha;\nend\n\nend\n  
cfar_os.m Ordered Statistical Constant False Alarm Rate Algorithm Implementation
Advantages: Good performance in multi-target detection;
Disadvantages: Increased false alarm probability in clutter edge regions.
function [ index, XT ] = cfar_os( xc, N, k, pro_N, PAD)\n% Assuming the echo follows a Gaussian distribution.\n%   Alpha assignment has some issues, a more complex high-order function\n\n%% Calculate alpha\n% syms alpha PFA;\n% PFA(alpha)=gamma(N-1).*gamma(N-k+alpha-1)./gamma(N-k-1)./gamma(N+alpha-1);\n% [alpha,~,~]=solve(PFA(alpha)==PAD,'ReturnConditions', true) ;\n\nalpha=N.*(PAD.^(-1./N)-1);\n\nindex=1+N/2+pro_N/2:length(xc)-N/2-pro_N/2;\nXT=zeros(1,length(index));\n\nfor i=index\n    cell_left=xc(1,i-N/2-pro_N/2:i-pro_N/2-1);\n    cell_right=xc(1,i+pro_N/2+1:i+N/2+pro_N/2);\n    cell_all=cat(2,cell_left,cell_right);\n    cell_sort=sort(cell_all);\n\n    Z=cell_sort(1,k);\n\n    XT(1,i-N/2-pro_N/2)=Z.*alpha;\nend\n\nend\ncfar_tc.m Implementation of Constant False Alarm Rate (CFAR) algorithm for clutter plots  
Advantage: By increasing the cumulative frames, the background noise power of clutter approaches the theoretical value.
Disadvantage: It has a self-masking effect on slow moving targets.
function [ index, XT, xc_tpn ] = cfar_tc( xc, xc_tp, N, pro_N, PAD)\n% Assuming the echo follows a Gaussian distribution.\n%\nxc_tc=zeros(1,length(xc));\nalpha=N.*(PAD.^(-1./N)-1);\nxc_tc(1,2:end-1)=xc_tp(1,2:end-1)./3+xc(1,1:end-2)./3+xc(1,3:end)./3;\nxc_tpn=xc_tp(1,1:end)./2+xc(1,1:end)./2;\nxc_tc(1,1)=xc_tpn(1,1);\nxc_tc(1,end)=xc_tpn(1,end);\n\nindex=1+N/2+pro_N/2:length(xc)-N/2-pro_N/2;\nXT=zeros(1,length(index));\n\nfor i=index\n    cell_left=xc_tc(1,i-N/2-pro_N/2:i-pro_N/2-1);\n    cell_right=xc_tc(1,i+pro_N/2+1:i+N/2+pro_N/2);\n    Z=(sum(cell_left)+sum(cell_right))./N;\n\n    XT(1,i-N/2-pro_N/2)=Z.*alpha;\nend\n\nend\n  
Section Five: CFAR Algorithm in Ideal Situation
① Multi-object detection performance is excellent, almost unaffected by each other;
② Complex clutter edge regions, with rapid changes, maintaining a stable false alarm rate;
③ Slow moving targets self-masking is improved;
④ The algorithm is practical, has moderate complexity, and theoretically feasible.

## Images

![(Constant False Alarm Ratio Detection)Classic CFAR algorithm analysis and performance map display [with MATLAB code]](01.jpg)
![(Constant False Alarm Ratio Detection)Classic CFAR algorithm analysis and performance map display [with MATLAB code]](02.jpg)
![(Constant False Alarm Ratio Detection)Classic CFAR algorithm analysis and performance map display [with MATLAB code]](03.jpg)
![(Constant False Alarm Ratio Detection)Classic CFAR algorithm analysis and performance map display [with MATLAB code]](04.jpg)
![(Constant False Alarm Ratio Detection)Classic CFAR algorithm analysis and performance map display [with MATLAB code]](05.jpg)
![(Constant False Alarm Ratio Detection)Classic CFAR algorithm analysis and performance map display [with MATLAB code]](06.jpg)

<!-- PAYMENT_BLOCK -->

## Get this dataset

- Payment link (Stripe): [Buy now](https://buy.stripe.com/3cs8yP7sY87d0vu9AB)
- Payment QR code: ![Payment QR code](pay.png)

<!-- /PAYMENT_BLOCK -->
