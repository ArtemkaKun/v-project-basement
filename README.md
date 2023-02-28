# What?

This repo contains universal minimum GitHub CI scripts and issue templates for V project.
After small modifications (specific to your project) you end up with "must-have" CI checks for
commits/PRs and nice bug/feature request templates. You will spend no time on DevOps stuff
and rush directly into the development process.

> **Note**
>
> I recommend using this basement for every V project, that will be stored on GitHub.

## Details

### `new-changes-validation` GitHub Actions workflow

The idea of this workflow is very simple - validate incoming changes,
so you make sure that changes, you are about to push to the main branch, work.
The philosophy of the workflow is also simple:

1. Code works
2. Code beautiful
3. Code fast

#### Triggers

- push (on the main branch)
- pull request

#### Flow

```mermaid
graph TD
	autofree_node[autofree]
	skip_unused_node[skip-unused]

	subgraph Simple_build_step[Simple build]
		autofree_node
		skip_unused_node
	end

	autofree_node_1[autofree]
	skip_unused_node_1[skip-unused]
	prod_node[prod]
	gcc_node[GCC]
	clang_node[Clang]

	subgraph Different_compilers_step[Different compilers]
		autofree_node_1
		skip_unused_node_1
		prod_node
		gcc_node
		clang_node
	end

	subgraph Cross_compilation_step[Cross compilation]
    subgraph Compile_from_system[Compiles from]
      direction LR
		  linux_base[Linux]
		  windows_base[Windows]
      macos_base[macOS]
    end

    subgraph Compile_to_system[Compiles to]
      direction LR
      linux_out[Linux]
		  windows_out[Windows]
      macos_out[macOS]
    end

    linux_base --> windows_out
    linux_base -.-> |only with native backend|macos_out

    windows_base --> linux_out
    windows_base -.-> |only with native backend|macos_out

    macos_base --> windows_out
    macos_base --> linux_out
	end

	style parallel_exec1 stroke:blue,stroke-width:2px,stroke-dasharray: 5 5
	style parallel_exec2 stroke:blue,stroke-width:2px,stroke-dasharray: 5 5
	style parallel_exec3 stroke:blue,stroke-width:2px,stroke-dasharray: 5 5
	style parallel_exec4 stroke:blue,stroke-width:2px,stroke-dasharray: 5 5

	style all_success1 stroke:green,stroke-width:2px,stroke-dasharray: 5 5
	style all_success2 stroke:green,stroke-width:2px,stroke-dasharray: 5 5
	style all_success3 stroke:green,stroke-width:2px,stroke-dasharray: 5 5
	style all_success4 stroke:green,stroke-width:2px,stroke-dasharray: 5 5

	start_node((Start))
	--> Simple_build_step
	--> tests_node[Tests]
	--> Different_compilers_step
	--> parallel_exec1{{Parallel execution}} --> clang_sanitizers_node[Clang sanitizers checks] & gcc_sanitizers_node[GCC sanitizers checks] & valgrind_node[Valgrind checks] --> all_success1{{All success}}
	--> fuzzer_node[Fuzzer testing]
	--> parallel_exec2{{Parallel execution}} --> Cross_compilation_step & Vab_compilation_node[Vab compilation] --> all_success2{{All success}}
	--> parallel_exec3{{Parallel execution}} --> format_node[Format check] & documentation_node[Documentation check] & vet_node[Vet check] --> all_success3{{All success}}
	--> parallel_exec4{{Parallel execution}} --> performance_regression_node[Performance regression check] & memory_regression_node[Memory regression check] --> all_success4{{All success}}
	--> end_node((End))
```

#### TODO
- [ ] Valgrind check
- [ ] Fuzzer testing
- [ ] Linux to macOS cross compilation
- [ ] Windows to other platforms cross compilation
- [ ] macOS to other platforms cross compilation
- [ ] Performance regression check
- [ ] Memory regression check

### Issue templates

#### `Bug` issue template

![Bug issue template example](https://user-images.githubusercontent.com/36485221/219885209-8343f5cf-fbab-428f-8090-f7b1979a5c62.png)

#### `Feature request` issue template

![Feature request issue template example](https://user-images.githubusercontent.com/36485221/219885262-3f770c2b-a51a-4c95-8954-43ff97fc792d.png)

# How?

## How to use it in my project?

Just copy-paste the `.github` folder to your V project.

# Why?

## Why does one need to use this basement?

It will help to improve the quality of your code and development process for any V project.
To decide if you need to use this basement, ask yourself this question
"Would my project benefit from automatic compilation, no memory leaks, and clean code checks?"

## Why GitHub Action jobs in `new-changes-validation` are not parallel?

I decided to make a checking process sequential because of 2 reasons:

1. Make a developer focus on the right thing to fix.
It will be at least strange to fix code format and write documentation
when your code doesn't compile. Steps that should run in parallel - run in parallel.
2. Don't load runners with unneeded work.
Don't sanitize your code if it doesn't compile, so your runner can do something more valuable
(maybe checking another project). This is not that clear when you use free GitHub runners,
but it's very clear when you have limited self-hosted runners.

# Thanks

- V community - for cool programming language and CI scripts
- @ulises-jeremias for issue templates
