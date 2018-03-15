# Machine-learning

import math
import random
import  numpy as np
import matplotlib.pyplot as plt
from sklearn import datasets

random.seed(0)

# 生成[a,b]区间内的随机数
def rand(a, b):
    return (b - a) * random.random() + a

# 生成m x n的矩阵
def make_matrix(m, n, fill=0.0):
    mat = []
    for i in range(m):
        mat.append([fill] * n)
    return mat

# 激活函数sigmoid
def sigmoid(x):
    return 1.0 / (1.0 + math.exp(-x))

# 激活函数的倒数
def sigmoid_derivative(x):
    return x * (1 - x)

# 定义BP神经网络
class BPNeuralNetwork:
    def __init__(self):
        self.input_n = 0
        self.hidden_n = 0
        self.output_n = 0
        self.input_cells = []
        self.hidden_cells = []
        self.output_cells = []
        self.input_weights = []
        self.output_weights = []
        self.input_correction = []
        self.output_correction = []

    def setup(self, ni, nh, no):
        # ANN的层数
        self.input_n = ni + 1
        self.hidden_n = nh
        self.output_n = no
        # 初始化神经元
        self.input_cells = [1.0] * self.input_n
        self.hidden_cells = [1.0] * self.hidden_n
        self.output_cells = [1.0] * self.output_n
        # 生成连接权值
        self.input_weights = make_matrix(self.input_n, self.hidden_n)
        self.output_weights = make_matrix(self.hidden_n, self.output_n)
        # 随机初始化连接权值
        for i in range(self.input_n):
            for h in range(self.hidden_n):
                self.input_weights[i][h] = rand(-0.2, 0.2)
        for h in range(self.hidden_n):
            for o in range(self.output_n):
                self.output_weights[h][o] = rand(-2.0, 2.0)
        # 初始化校正矩阵
        self.input_correction = make_matrix(self.input_n, self.hidden_n)
        self.output_correction = make_matrix(self.hidden_n, self.output_n)

    # 输入前向传播
    def predict(self, inputs):
        # 输入层神经元
        for i in range(self.input_n - 1):
            self.input_cells[i] = inputs[i]
        # 隐层神经元
        for j in range(self.hidden_n):
            total = 0.0
            for i in range(self.input_n):
                total += self.input_cells[i] * self.input_weights[i][j]
            self.hidden_cells[j] = sigmoid(total)
        # 输出层神经元
        for k in range(self.output_n):
            total = 0.0
            for j in range(self.hidden_n):
                total += self.hidden_cells[j] * self.output_weights[j][k]
            self.output_cells[k] = sigmoid(total)
        return self.output_cells[:]

    # 误差后向传播
    def back_propagate(self,case,label,learn,correct):
        self.predict(case)
        # 计算输出误差
        output_deltas = [0.0]*self.output_n
        for i in range(self.output_n):
            error = label[i] - self.output_cells[i]
            output_deltas[i] = error * sigmoid_derivative(self.output_cells[i])
        # 计算隐层误差
        hidden_deltas = [0.0] * self.hidden_n
        for j in range(self.hidden_n):
            error = 0
            for o in range(self.output_n):
                error += output_deltas[i] * self.output_weights[j][o]
            hidden_deltas[j] = error * sigmoid_derivative(self.hidden_cells[j])

        # 更新隐层到输出层权值
        for j in range(self.hidden_n):
            for o in range(self.output_n):
                change = output_deltas[o] * self.hidden_cells[j]
                self.output_weights[j][o] += learn * change + correct * self.output_correction[j][o]
                self.output_correction[j][o] = change

        # 更新输入层到隐层权值
        for i in range(self.input_n):
            for j in range(self.hidden_n):
                change = hidden_deltas[j] * self.input_cells[i]
                self.input_weights[i][j] += learn * change + correct * self.input_correction[i][j]
                self.input_correction[i][j] = change

        # 计算输出误差
        error = 0.0
        for o in range(self.output_n):
            error += 0.5 * (label[o] - self.output_cells[o]) ** 2
        return error

    # 训练函数
    def train(self, cases, labels, learn = 0.05, correct = 0.1, limit = 10000):
        error = [0.0] * limit
        for o in range(limit):
            error[o] = 0.0
            for i in range(len(cases)):
                label = labels[i]
                case = cases[i]
                error[o] += self.back_propagate(case, label, learn, correct)
        # 绘制误差曲线
        plt.figure(1, dpi=50)
        x = np.arange(0, limit)
        plt.plot(x, error, label = "error")
        plt.title("Train error")
        plt.xlabel("Train time")
        plt.ylabel("Error")
        plt.legend()
        plt.show()

    # 测试函数
    def test(self):
        iris = datasets.load_iris()
        cases = np.array(iris.data)
        labels = np.array(iris.target)
        #labels = labels1[:, None]
        # cases = [
        #     [0, 0, 1],
        #     [1, 0, 1],
        #     [0, 1, 1],
        #     [1, 1, 1],
        #     [1, 0, 0],
        # ]
        # labels = [[0], [1], [1], [0], [0]]
        self.setup(4, 5, 1)
        self.train(cases, labels, 0.05, 0.1, 10000)
        for case in cases:
            print(self.predict(case))

if __name__ == '__main__':
    nn = BPNeuralNetwork()
    nn.test()




