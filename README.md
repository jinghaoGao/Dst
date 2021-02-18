# Dst
SVM and BPNN main code
## I. 清空环境变量
clear all
clc
## II. 导入数据
load dst_yaoban.mat
 

##1角宽2CME初始速度3加速度4质量5能量6MPA7X-flare8耀斑能量通量9icme极值10
##dc+mc 1 2 3 4 8
##mc 123468
##quan 12345678
##dc 1234578
## 1. 随机产生训练集和测试集
n1 = randperm(size(dst_yaoban,1));
 
##
## 2. 训练集――161个样本
p_train = dst_yaoban(n1(1:161),[1,2,3,4,5,6,7,8]);
##p_train = dst_yaoban(n1(1:161),[1,2,3,4,5,7,8]);
##p_train = dst_yaoban(n1(1:161),[1,2,3,4,6,8]);
##p_train = dst_yaoban(n1(1:161),[1,2,3,4,8]);
t_train = dst_yaoban(n1(1:161),10);
 
##
## 3. 测试集――25个样本
p_test = dst_yaoban(n1(162:end),[1,2,3,4,5,6,7,8]);
##p_test = dst_yaoban(n1(162:end),[1,2,3,4,5,7,8]);##DC
##p_test = dst_yaoban(n1(162:end),[1,2,3,4,6,8]);##MC
##p_test = dst_yaoban(n1(162:end),[1,2,3,4,8]);##DC+MC
t_test = dst_yaoban(n1(162:end),10);
 
## III. 数据归一化
##
## 1. 输入集
[pn_train,inputps] = mapminmax(p_train');
pn_train = pn_train';
pn_test = mapminmax('apply',p_test',inputps);
pn_test = pn_test';
 
##
## 2. 输出集
[tn_train,outputps] = mapminmax(t_train');
tn_train = tn_train';
tn_test = mapminmax('apply',t_test',outputps);
tn_test = tn_test';
 
## IV. SVM模型创建/训练
##
## 1. 寻找最佳c参数/g参数
[c,g] = meshgrid(-10:0.5:10,-10:0.5:10);
[m,n] = size(c);
cg = zeros(m,n);
eps = 10^(-4);
v = 5;
bestc = 0;
bestg = 0;
error = Inf;
for i = 1:m
    for j = 1:n
        cmd = ['-v ',num2str(v),' -t 2',' -c ',num2str(2^c(i,j)),' -g ',num2str(2^g(i,j) ),' -s 3 -p 0.1'];
        cg(i,j) = svmtrain(tn_train,pn_train,cmd);
        if cg(i,j) < error
            error = cg(i,j);
            bestc = 2^c(i,j);
            bestg = 2^g(i,j);
        end
        if abs(cg(i,j) - error) <= eps && bestc > 2^c(i,j)
            error = cg(i,j);
            bestc = 2^c(i,j);
            bestg = 2^g(i,j);
        end
    end
end
 
##
## 2. 创建/训练SVM  
cmd = [' -t 2',' -c ',num2str(bestc),' -g ',num2str(bestg),' -s 3 -p 0.1'];
model = svmtrain(tn_train,pn_train,cmd);
 
## V. SVM仿真预测
[Predict_1,error_1,decision_values1] = svmpredict(tn_train,pn_train,model);
[Predict_2,error_2,decision_values2] = svmpredict(tn_test,pn_test,model);
##
## 1. 反归一化
predict_1 = mapminmax('reverse',Predict_1,outputps);
predict_2 = mapminmax('reverse',Predict_2,outputps);
 
##
## 2. 结果对比
result_1 = [t_train predict_1];
result_2 = [t_test predict_2];
## VI. 绘图
## num2str把数值转换成字符串
figure(1)
plot(1:length(t_train),t_train,'r-*',1:length(t_train),predict_1,'b:o')
grid on
legend('真实值','预测值')
xlabel('样本编号')
ylabel('dst强度')
string_1 = {'训练集预测结果对比';
           ['mse = ' num2str(error_1(2)) ' R = ' num2str(sqrt(error_1(3)))]};
title(string_1)
figure(2)
plot(1:length(t_test),t_test,'r-*',1:length(t_test),predict_2,'b:o')
grid on
legend('真实值','预测值')
xlabel('样本编号')
ylabel('dst强度')
string_2 = {'测试集预测结果对比';
           ['mse = ' num2str(error_2(2)) ' R = ' num2str(sqrt(error_2(3)))]};
title(string_2)
 
## VI. 绘图
figure(1)
plot(1:length(t_train),t_train,'r-*',1:length(t_train),predict_1,'b:o')
grid on
legend('真实值','预测值')
xlabel('样本编号')
ylabel('dst强度')
string_1 = {'训练集预测结果对比';
           ['mse = ' num2str(error_1(2)) ' R^2 = ' num2str(error_1(3))]};
title(string_1)
figure(2)
plot(1:length(t_test),t_test,'r-*',1:length(t_test),predict_2,'b:o')
grid on
legend('真实值','预测值')
xlabel('样本编号')
ylabel('dst强度')
string_2 = {'测试集预测结果对比';
           ['mse = ' num2str(error_2(2)) ' R^2 = ' num2str(error_2(3))]};
title(string_2)

## 绘图2
##SVM 预测值与观测值的对比图
figure(3);
x = (-450:0.01:0);
y = (-450:0.01:0);
hold on;
title('SVM全磁暴峰值预测','fontsize',12)
plot(t_test,predict_2,'k.','markersize',16);
plot(x,y,'k-','markersize',10);
xlabel('Observation Peak of magnetic storm （nT）','fontsize',14);
ylabel('Forecasting Peak of magnetic storm （nT）','fontsize',14);
set(gca,'Xtick',(-450:50:0),'Ytick',(-450:50:0));
axis square;
box on;
set(gca,'FontSize',12,'XAxisLocation','YAxisLocation');
hold off;
##SVM 训练的预测值与观测值的对比图
figure(4);
x = (-450:0.01:0);
y = (-450:0.01:0);
hold on;
title('SVM训练集全磁暴峰值预测','fontsize',12)
plot(t_train,predict_1,'k.','markersize',16);
plot(x,y,'k-','markersize',10);
xlabel('Observation Dst value（nT）','fontsize',14);
ylabel('Forecasting Dst value（nT）','fontsize',14);
set(gca,'Xtick',(-450:50:0),'Ytick',(-450:50:0));
axis square;
box on;
set(gca,'FontSize',12,'XAxisLocation','bottom','YAxisLocation');
hold off;

##英文的预测图
## 绘图2
##SVM 预测值与观测值的对比图
figure(4);
x = (-450:0.01:0);
y = (-450:0.01:0);
hold on;
title('SVM全磁暴峰值预测','fontsize',12)
plot(t_test,predict_2,'k.','markersize',16);
plot(x,y,'k-','markersize',10);
xlabel('Observation Peak of magnetic storm （nT）','fontsize',14);
ylabel('Forecasting Peak of magnetic storm （nT）','fontsize',14);
set(gca,'Xtick',(-450:50:0),'Ytick',(-450:50:0));
axis square;
box on;
set(gca,'FontSize',12,'XAxisLocation','YAxisLocation');
hold off;


## VII. BP神经网络
##
## 1. 数据转置
pn_train = pn_train';
tn_train = tn_train';
pn_test = pn_test';
tn_test = tn_test';
 
##
## 2. 创建BP神经网络
net = newff(pn_train,tn_train,10);
 
##
## 3. 设置训练参数
net.trainParam.epochs = 1000;
##net.trainParam.goal = 1e-3;
net.trainParam.goal = 1e-2;
net.trainParam.show = 10;
net.trainParam.lr = 0.1;
 
##
## 4. 训练网络
net = train(net,pn_train,tn_train);
 
##
## 5. 仿真测试
tn_sim = sim(net,pn_test);
 
##
## 6. 均方误差
E = mse(tn_sim - tn_test);
 
##
## 7. 决定系数
N = size(t_test,1);
R2=(N*sum(tn_sim.*tn_test)-sum(tn_sim)*sum(tn_test))^2/((N*sum((tn_sim).^2)-(sum(tn_sim))^2)*(N*sum((tn_test).^2)-(sum(tn_test))^2)); 
R=sqrt(R2) 
##
## 8. 反归一化
t_sim = mapminmax('reverse',tn_sim,outputps);
 
##
##9. 绘图
figure(5)
plot(1:length(t_test),t_test,'r-*',1:length(t_test),t_sim,'b:o')
grid on
legend('真实值','预测值')
xlabel('样本编号')
ylabel('dst强度')
string_3 = {'测试集预测结果对比(BP神经网络)';
           ['mse = ' num2str(E) ' R = ' num2str(sqrt(R2))]};
title(string_3)


## 绘图2
##BPNN 预测值与观测值的对比图
figure(6);
x = (-450:0.01:0);
y = (-450:0.01:0);
hold on;
title('BPNN全磁暴峰值预测','fontsize',12)
plot(t_test,t_sim','k.','markersize',16);
plot(x,y,'k-','markersize',10);
xlabel('Observation Peak of magnetic storm （nT）','fontsize',14);
ylabel('Forecasting Peak of magnetic storm （nT）','fontsize',14);
set(gca,'Xtick',(-450:50:0),'Ytick',(-450:50:0));
axis square;
box on;
set(gca,'FontSize',12,'XAxisLocation','YAxisLocation');
hold off;
