---
layout: post
title: "A minimal implementation of a deep learning compiler"
author: Job Hernandez Lara
tags: compilers 
---

version*: 0.9.0, initial version, draft


* Table of Contents
{:toc}

### Why I am writing this

So, I have been writing compilers for sometime and some of the compilers that interest me are deep learning compilers. I started reading research papers and slowly became more curious.

One bottleneck I experienced in the beginning of my compiler journey is how to use a parser generator. Once I learned to use a parser generator and use one to generate the parse tree I was able to tackle the compiler problem. Given a parse tree and an abstract syntax tree for it, you just figure out the solution in code like you would while developing a web backend.

Similarly, when I started investigating deep learning compilers I could not start working on one because I lacked understanding of how these compilers manipulate the computational graph. When I figured out how to do this I was able to figure out how to go from the computational graph to the generated C code.

So, in this post I would like to share my experience, including some code, so someone new to this can get a better idea of how to get the computational graph without having to investigate the TVM codebase.


### Why are deep learning compilers important?

It is my understanding that deep learning compilers are important because libraries and frameworks do not scale. It has been said by industry leaders that libraries and frameworks do not scale because the deep learning operators in these libraries are fine tuned to each computer architecture; as a result, when deep learning and hardware architectures, each new deep learning operator has to be implemented for each computer architecture.

In other words, deep learning compilers are important because they allow engineers to cope with the diversity of hardware.[^1] Libraries are slow in adapting with new hardware architectures so it is hard for these libraries to utilize all the power of these architectures.

In contrast, deep learning compilers can target multiple hardware architectures.

In her article[^2], Chip Huyen talks about how edge computing is becoming more important. So, if you want your models to run on people’s cell phones and computers then you will need to target different architectures.

### A simplified look into how deep learning compilers work

Deep learning compilers are compilers that take a neural network model defined in Pytorch or another framework, and compile this model to optimized code. The point for this is to improve the inference time of a given model. 

As discussed above, Deep learning compilers make targeting different computer architectures feasible.

Deep learning compilers have a frontend and backend.

The following is based on this  deep learning survey.[^1] You should read it to get a better idea.

The frontend of a deep learning compiler consists of a high level intermediate representation (IR). This IR is a directed acyclic graph (DAG). The nodes in this graph represent the deep learning operators such as convolution. On the other hand, the edges represent tensors. Several optimizations that are independent of the hardware are applied to this high level IR.

In addition there's also a low level IR, which is part of the backend, to which target dependent optimizations are applied. One type of low level IR is based on Halide. When a Halide based IR is used, one separates the computation from the schedule. Given a computation, one tries different schedulers to get best performance. This is the approach taken by TVM. In addition, the backend is also responsible for generating the actual code for the different hardware architectures.

### My experience writing a minimal deep learning compiler

The Pytorch TVM frontend is defined [here](https://github.com/apache/tvm/blob/main/python/tvm/relay/frontend/pytorch.py#L5282) in line 5282. If you scroll down to line [5360](https://github.com/apache/tvm/blob/main/python/tvm/relay/frontend/pytorch.py#L5360) you will also notice how TVM gets the computational graph of a given Pytorch model.

Based on TVM’s implementation, I figured out the following implementation which takes a convolutional layer defined in Pytorch and gets the computational graph and generates C code.

You can check out my compiler [here](https://github.com/Jobhdez/convolution-layer-to-C).

Here is how I manipulated the computational graph. The idea of my solution was to combine the graph information obtained from `torch.jit.trace` and `torch.fx.symbolic_trace` to get the appropriate information about the graph such as operators in the layers and actual tensors.

Although I do not construct a DAG, I still created an abstract syntax tree to represent the deep learning model. 

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
	print(f'hello: {type(node)}')
	if isinstance(node, Conv2dNode):
       	 
    	c_str = ""
    	c_str = c_str = '#include "runtime.c"\n'
    	c_str = c_str + "#include <stdio.h>\n\n"

    	bias = str(node.bias)
    	input_h = str(node.input_height)
    	input_w = str(node.input_width)
    	filter_h = str(node.filter_height)
    	filter_w = str(node.filter_width)
    	batch_s = str(node.batch_size)
    	channels = str(node.channels)

    	tensor = torch_tensor_to_c(node.input_tensor)
    	weight = torch_tensor_to_c(node.weight)
   	 
    	c_str = c_str + "\nint main() {\n\n"
    	c_str = c_str + f'int batch_size = {str(int(batch_s))};\n\n'
    	c_str = c_str + f'int channels = {str(int(channels))};\n\n'
    	c_str = c_str + f'int input_height = {str(int(input_h))};\n\n'
    	c_str = c_str + f'int input_width = {str(int(input_w))};\n\n'
    	c_str = c_str + f'int filter_height = {str(int(filter_h))};\n\n'
    	c_str = c_str + f'int filter_width = {str(int(filter_w))};\n\n'

    	c_str = c_str + f'float input_data[{batch_s}][{channels}][{input_h}][{input_w}] = {tensor};\n\n'
    	c_str = c_str + f'float weight[{batch_s}][{channels}][{filter_h}][{filter_w}] = {weight};\n\n'
    	c_str = c_str + f'float bias = {bias};\n'
    	c_str = c_str + f'float output[{batch_s}][{channels}][{filter_h}][{filter_h}];\n\n'
    	c_print = "convolution(input_data, weight, &bias, batch_size, channels, input_height, input_width, filter_height, filter_width, output);\n\n"
    	c_print = c_print + "for (int i = 0; i < batch_size; ++i) {\n"
    	c_print = c_print + "  for (int j = 0; j < 1; ++j) {\n"
    	c_print = c_print + " 	for (int k = 0; k < 3; ++k) {\n"
    	c_print = c_print + "   	for (int l = 0; l < 3; ++l) {\n"
    	c_print = c_print + '     	printf("%f ", output[i][j][k][l]);\n'
    	c_print = c_print + "   	}\n"
    	c_print = c_print + " 	}\n"
    	c_print = c_print + "  }\n"
    	c_print = c_print + "}\n"
   
    	c_str = c_str + c_print + 'return 0;\n}'
    	return c_str


           	 
def torch_tensor_to_c(tensor):
	c_array = tensor.numpy()
	c_array = c_array.tolist()
	c_array = str(c_array)
	print(c_array)
	c_array = c_array.replace('[', '{').replace(']', '}')
	return c_array

```

And here is the example that I ran:

```python
# == utils for the example ==

def write_file(c_program, file_name):
	with open(file_name, "w") as f:
    	f.write(c_program)

def test_my_conv_fn(input_tensor, net):
	module = torch.jit.trace(net, input_tensor)
	state_dict = module.state_dict()
	weight = state_dict['conv.weight']
	bias = state_dict['conv.bias']
	saved_module = module.save("test_conv.pth")
	load_module = torch.jit.load("test_conv.pth")

	with torch.no_grad():
    	torch_output = load_module(input_tensor)

	my_conv_output = convolution_torch(input_tensor, weight, bias)

	print(f'torch_output: {torch_output}\n my_conv_output: {my_conv_output}')

# == Example code for the compilation of a conv layer to C ===

class Net(nn.Module):
	def __init__(self):
    	super().__init__()
    	self.conv = nn.Conv2d(1, 1, 3).float()

	def forward(self, x):
    	return self.conv(x)

net = Net()
ones = np.ones((1,1,3,3), dtype=np.float32)
input_tensor = torch.tensor(ones, dtype=torch.float32)

nn_ast = torch_to_ast(net, input_tensor)
c_program = to_c(nn_ast[0])

print("\ntesting my conv fn....")
test_my_conv_fn(input_tensor, net)

print("\n\nwriting generated C file...")
write_file(c_program, "../backend/conv.c")
```

So, given a model such as the following:

```
class Net(nn.Module):
	def __init__(self):
    	super().__init__()
    	self.conv = nn.Conv2d(1, 1, 3).float()

	def forward(self, x):
    	return self.conv(x)
```

My compiler generates the C code. You can take a look at the generate code for this example [here](https://github.com/Jobhdez/convolution-layer-to-C/blob/main/src/backend/conv.c).

### Summary

Hopefully, you have a better idea of how, at least, a minimal deep learning compiler works in practice. I left out the optimizations but you can learn more about the optimizations from the “The Deep Learning Compiler: A Comprehensive Survey” that is listed in the references. The basic idea is that given a deep learning model defined in Pytorch, the first thing is to build the computational graph consisting of deep learning operators such as convolution and tensors. Once you have the graph then you apply machine independent optimizations. After this you  build a low level IR to which machine dependent optimizations are applied and finally you generate the code.

### Some relevant further reading

If you want to explore this further I recommend the repo: [Awesome Tensor Compilers](https://github.com/merrymercy/awesome-tensor-compilers).

Specifically, from that list, the Tiramisu, TVM, and Hidet papers are very good.

### References

[^1]: [The Deep Learning Compiler](https://arxiv.org/pdf/2002.03794.pdf)
[^2]: [A Friendly Introduction to Machine Learning Compilers and Optimizers](https://huyenchip.com/2021/09/07/a-friendly-introduction-to-machine-learning-compilers-and-optimizers.html)
