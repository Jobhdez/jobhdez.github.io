---
layout: post
title: "A minimal implementation of a deep learning compiler"
author: Job Hernandez Lara
tags: compilers 
---

* Table of Contents
{:toc}

### Introduction

Deep learning compilers are the key to deploy neural nets on different devices and architectures. And with the end of Moore's Law, they will be increasingly important. It is my understanding that deep learning compilers are important because libraries and frameworks do not scale. It has been said by industry leaders that libraries and frameworks do not scale because the deep learning operators in these libraries are fined tuned to each computer architecture; as a result, when there is a new architecture all the operators need to be re-implemented for this architecture.

In other words, deep learning compilers are important because they allow engineers to cope with the diversity of hardware.[^1] Libraries are slow in adapting with new hardware architectures so it is hard for these libraries to utilize all the power of these architectures; moreover, deep learning compilers enable graph level optimizations and operator level optimization.

In her article[^2], Chip Huyen talks about how edge computing is becoming more important. So, if you want your models to run on people’s cell phones and computers then you will need to target different architectures.


### Computational Graphs

A neural network is a computational graph whose nodes are deep learning operators such as convolution or tensors and the edges are dependencies among them. Technically speaking, a computational graph is a directed acyclic graph; for example, the expression:

$$ a + a \times (b - c) + (b - c) \times d $$

is represented as:

```         
	   +
        /     \
       +        *
     /  \      / \
    /     *   /   d
    \   /  \ /
      a    -
          / \
         b   c
```

Notice that common sub expressions are only used once. DAGs as these types of graphs are called, work the same way in regular compilers and deep learning compilers. Sub expressions are eliminated because it enables better performance because give two common sub expressions, the sub expression is only computed once.

If you take a look at the TVM[^3] documentation, it mentions that a lot of the optimizations are DAG based.

### How do deep learning compilers?

Deep learning compilers take a DAG consisting of deep learning operators such as convolution and relu and lowers this graph to low level code such as LLVM IR. Since in a deep learning compiler you do not need to optimize kernels manually, one can target different architectures which is why deep learning compilers scale better.

### A simplified look into how deep learning compilers work

The following is based on this  deep learning survey.[^1] You should read it to get a better idea.

The frontend of a deep learning compiler consists of a high level intermediate representation (IR). This IR is a directed acyclic graph (DAG). The nodes in this graph represent the deep learning operators such as convolution. On the other hand, the edges represent tensors. Several optimizations that are independent of the hardware are applied to this high level IR. In other words, this graph is indeed the computational graph we need to get from a given neural net model. Once we have this graph, we have access to the deep learning operators such as convolution and the tensors such as weights, and biases.

In addition there's also a low level IR, which is part of the backend, to which target dependent optimizations are applied. One type of low level IR is based on Halide. When a Halide based IR is used, one separates the computation from the schedule. Given a computation, one tries different schedules to get best performance. A schedule is a type of optimization that can be applied such as tiling or vectorization. The idea is that by applying a series of schedules one can get better performance. This is the approach taken by TVM. In addition, the backend is also responsible for generating the actual code for the different hardware architectures.

#### Example

TVM[^4] is a state of the art deep learning compiler it compiles deep learning models defined in frameworks, e.g., Pytorch to different architectures. This compiler works by representing the computational graph in an IR called Relay IR; for example, consider the example in the TVM documenation[^5]:

```
x = relay.var("x"
v1 = relay.log(x)
v2 = relay.add(v1 v1)
f = relay.Function([x], v2)
```

This example corresponds to the following computation graph

```

            var
	     |
	    log
	   /  \
           \  /
   result<-add
```

In the TVM paper[^4], it talks about the type of optimizations that are applied to this Relay IR. According to the TVM paper, some of these optimizations consist of *operation fusion* whereby multiple operators are fused into a single kernel. Operation fusion improves execution time because intermediate computations do not get saved in memory. In addition to operation fusion, *constant fold* is applied as well. Constant fold is an optimization that precomputes parts of the graph.

Before the Relay graph gets lowered to low level code, the Relay operators -- e.g., relay.conv2d -- get mapped to a high level tensor expression language. The corresponding LLVM IR gets generated from this.

#### A minimal implementation of a deep learning compiler


I built a deep learning compiler[^5] before having an idea of how the deep learning model graph was represented internally in the compiler but I think I guessed partly right. Here is the code that takes the following model and generates an AST

```python
class Net(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv = nn.Conv2d(1, 1, 3).float()

    def forward(self, x):
        return self.conv(x)
```
Internally, in the AST the Conv2d is represented with this node:

```python
class Conv2dNode:
    def __init__(self, input_tensor, weight, bias, input_height, input_width, filter_height, filter_width, batch_size, channels):
        self.input_tensor = input_tensor
        self.weight = weight
        self.bias = bias
        self.input_height = input_height
        self.input_width = input_width
        self.filter_height = filter_height
        self.filter_width = filter_width
        self.batch_size = batch_size
        self.channels = channels
```

If you take a look at TVM's Pytorch frontend which is defined [here](https://github.com/apache/tvm/blob/main/python/tvm/relay/frontend/pytorch.py#L5282) in line 5282. If you scroll down to line [5360](https://github.com/apache/tvm/blob/main/python/tvm/relay/frontend/pytorch.py#L5360) you will also notice how TVM gets the computational graph from a given Pytorch model.

Based on TVM’s implementation, I figured out the following implementation which takes a convolutional layer defined in Pytorch and gets the computational graph and generates C code.

Here is how I manipulated the computational graph. The idea of my solution was to combine the graph information obtained from `torch.jit.trace` and `torch.fx.symbolic_trace` to get the appropriate information about the graph such as operators in the layers and actual tensors.

I also had to experiment with TVM's API and read the Pytorch documentation but once I was able to manipulate the operators and tensors I only had figure out how a convolution operator is implemented and then write it in C as part of the runtime.

I am not 100 percent sure if this is the approach taken by TVM. I was looking at the TVM compiler back-end code and some neural net operators are implemented but they looked like constructors instead of the actual computation. I will have to research this more.

Anyways, although I did not construct a DAG, I still created an abstract syntax tree to represent the deep learning model. 

```python
def torch_to_ast(net, input_tensor):
	module = torch.jit.trace(net, input_tensor)
	gm = torch.fx.symbolic_trace(net)
	layers = get_layers(gm)
	nodes = ShapeProp(gm)
	nodes.propagate(input_tensor)
    
	tensor_data = []
	for node in gm.graph.nodes:
    	tensor_data.append({'name': node.name, 'dtype': node.meta['tensor_meta'].dtype, 'shape': node.meta['tensor_meta'].shape, 'tensor': node.meta['tensor_meta'].data})

	layers_lst = [list(layer) for layer in layers]
	print(layers_lst)
	layers = layers_lst[1:]
	print(layers)
	layers = [{'name': layer[0], 'nn_obj': layer[1]} for layer in layers]
	print(layers)
	print(type(layers[0]['nn_obj']))
	ast_nodes = []
	for layer in range(len(layers)):
    	nn_obj = layers[layer]['nn_obj']
    	print(nn_obj)
    	print(type(nn_obj))
    	if isinstance(nn_obj, torch.nn.modules.conv.Conv2d):
        	input_tensor = None
        	weight_tensor = None
        	for tensor in tensor_data:
            	if tensor['name'] == 'x':
                	input_tensor = tensor['tensor']
            	elif tensor['name'] == layers[layer]['name']:
                	input_name = tensor['name']
                	weight_tensor = tensor['tensor']
        	state_dict = module.state_dict()
        	bias_name = input_name + ".bias"
        	weight = state_dict[input_name + ".weight"]
        	bias_tensor = state_dict[bias_name]
        	batch_size, channels, input_height, input_width = input_tensor.shape
        	_,_, filter_height, filter_width = weight.shape
        	bias = None
        	if len(bias_tensor) == 1 and isinstance(float(bias_tensor[0]), float):
            	bias = float(bias_tensor[0])
        	else:
            	bias = bias_tensor
        	ast_nodes.append(Conv2dNode(input_tensor, weight, bias, input_height, input_width, filter_height, filter_width, batch_size, channels))
	return ast_nodes

def get_layers(graph):
    
	return list(graph.named_modules())
```

And here is how I generated the C code.

```python
            
def to_c(node):
    if isinstance(node, Conv2dNode):
        c_str = """
#include "runtime.c"
#include <stdio.h>

int main() {
"""
        bias = str(node.bias)
        input_h = str(node.input_height)
        input_w = str(node.input_width)
        filter_h = str(node.filter_height)
        filter_w = str(node.filter_width)
        batch_s = str(node.batch_size)
        channels = str(node.channels)

        tensor = torch_tensor_to_c(node.input_tensor)
        weight = torch_tensor_to_c(node.weight)
        
        c_str += f"    int batch_size = {str(int(batch_s))};\n"
        c_str += f"    int channels = {str(int(channels))};\n"
        c_str += f"    int input_height = {str(int(input_h))};\n"
        c_str += f"    int input_width = {str(int(input_w))};\n"
        c_str += f"    int filter_height = {str(int(filter_h))};\n"
        c_str += f"    int filter_width = {str(int(filter_w))};\n"
        
        c_str += f"    float input_data[{batch_s}][{channels}][{input_h}][{input_w}] = {tensor};\n"
        c_str += f"    float weight[{batch_s}][{channels}][{filter_h}][{filter_w}] = {weight};\n"
        c_str += f"    float bias = {bias};\n"
        c_str += f"    float output[{batch_s}][{channels}][{filter_h}][{filter_h}];\n"
        
        c_str += """
    convolution(input_data, weight, &bias, batch_size, channels, input_height, input_width, filter_height, filter_width, output);

    printf("%f", output[0][0][0][0]);
       
    return 0;
}
"""
        return c_str

def torch_tensor_to_c(tensor):
    c_array = tensor.numpy().tolist()
    c_array = str(c_array).replace('[', '{').replace(']', '}')
    return c_array
```

And here is the C code that my compiler generates:

```c
#include "runtime.c"
#include <stdio.h>

int main() {
    int batch_size = 1;
    int channels = 1;
    int input_height = 3;
    int input_width = 3;
    int filter_height = 3;
    int filter_width = 3;
    float input_data[1][1][3][3] = {{{{1.0, 1.0, 1.0}, {1.0, 1.0, 1.0}, {1.0, 1.0, 1.0}}}};
    float weight[1][1][3][3] = {{{{0.1206025704741478, 0.2804994285106659, 0.10350386798381805}, {-0.31444603204727173, -0.3109368085861206, -0.22286033630371094}, {-0.12215566635131836, 0.2765805721282959, 0.25621065497398376}}}};
    float bias = -0.24808375537395477;
    float output[1][1][3][3];

    convolution(input_data, weight, &bias, batch_size, channels, input_height, input_width, filter_height, filter_width, output);

    printf("%f", output[0][0][0][0]);
       
    return 0;
}
```

### How to improve it

I have plans to keep working on this compiler but I need to study TVM more. Next, I want to compile a VGG block to C code. Now, we can manually write some of the optimizations that compilers use to help GCC generate faster code. And we can also generate vectorized C code. 

### Summary

Hopefully, you have a better idea of how, at least, a minimal deep learning compiler works in practice. I left out the optimizations but you can learn more about the optimizations from the “The Deep Learning Compiler: A Comprehensive Survey” that is listed in the references. The basic idea is that given a deep learning model defined in Pytorch, the first thing to do is build the computational graph consisting of deep learning operators such as convolution and the tensors. Once you have the graph then you apply machine independent optimizations. After this you  build a low level IR to which machine dependent optimizations are applied and finally you generate the code.

### Some relevant further reading

If you want to explore this further I recommend the repo: [Awesome Tensor Compilers](https://github.com/merrymercy/awesome-tensor-compilers).

Specifically, from that list, the Tiramisu, TVM, and Hidet papers are very good.

### References

[^1]: [The Deep Learning Compiler](https://arxiv.org/pdf/2002.03794.pdf)
[^2]: [A Friendly Introduction to Machine Learning Compilers and Optimizers](https://huyenchip.com/2021/09/07/a-friendly-introduction-to-machine-learning-compilers-and-optimizers.html)
[^3]: [Relay IR](https://tvm.apache.org/docs/v0.9.0/arch/relay_intro.html)
[^4]: [The TVM Paper](https://www.usenix.org/system/files/osdi18-chen.pdf)
[^5]: [My Deep Learning compiler](https://github.com/Jobhdez/Convy)