# Neural Baseline and GECA for Grounded SCAN

This repository contains a multi-modal neural sequence-to-sequence model with a CNN to parse a world state and joint attention over input instruction sequences and world states.
This model is detailed in the grounded SCAN paper, and graphically depicted in the following image:

![Model](https://raw.githubusercontent.com/LauraRuis/multimodal_seq2seq_gSCAN/master/documentation/model_bahdanau.png?token=AGNMPFBNLBC3MIS3SQKJAXK6N7NYW)

## TL;DR

Find all commands to reproduce the experiments in `all_experiments.sh` containing the used parameters and seeds.
For a detailed file with all parameters, seeds, and other logging per training run, see `documentation/training_logs/`.
Before trainig unzip the data in folder `data`. For a demo on training a small model on a dummy dataset, refer to the last section of this readme headed 'demo'.

## Getting Started

Make a virtualenvironment that uses Python 3.7 or higher:

```virtualenv --python=/usr/bin/python3.7 <path/to/virtualenv>```

Activate the environment and install the requirements with a package manager:

```{ source <path/to/virtualenv>/bin/activate; python3.7 -m pip install -r requirements; }```

Note that this repository depends on the grounded SCAN implementation to load the dataset from a dataset.txt with the function `GroundedScan.load_dataset_from_file()`.
Before actually training models, unzip the data you want to use in the folder `data`.

### Alternative way of loading data
In the folder `read_gscan` there is a separate `README.md` and code to read about how to prepare the data for a computational model in a way independent from the code in `GroundedScan`.


## Contents

Sequence to sequence models for Grounded SCAN.

### Training

To train a model on a grounded SCAN dataset with a simple situation representation (for more information LINK TO gSCAN / paper), run:

    python3.7 -m seq2seq --mode=train --data_directory=<path/to/folder/with/dataset.txt/> --output_directory=<path/where/models/will/be/saved> --attention_type=bahdanau --max_training_iterations=200000

This will train a model and save the results in `output_directory`.

### Testing

To test with a model and obtain predictions, run the following command.

    python3.7 -m seq2seq --mode=test --data_directory=<path/to/folder/with/dataset.txt/> --attention_type=bahdanau --no_auxiliary_task --conditional_attention --output_directory=<path/containing/trained/models> --resume_from_file=adverb_k_1_run_3/model_best.pth.tar --splits=test,dev,visual,visual_easier,situational_1,situational_2,contextual,adverb_1,adverb_2 --output_file_name=predict.json --max_decoding_steps=120

NB: the output .json file generated by this can be passed to the error analysis or execute commands mode in [the grounded SCAN repo](https://github.com/LauraRuis/groundedSCAN). The repository at that link also contains a file `example_prediction.json` with 1 data example prediction as generated with the test mode of this repository.

## Important arguments

- `max_decoding_steps`: reflect max target length in data
- `k`: how many examples to add to the training set that contain 'cautiously' as an adverb
- `max_testing_examples`: testing is slow, because it is only implemented for batch size 1, so to speed up training use a small amount of testing examples for evaluation on the development set during training.


## Reproducing experiments from grounded SCAN paper

See file `all_experiments.sh` for all commands that were run to train the models for the grounded SCAN paper, as well as the commands to test trained models.


## Reproducability: Hyperparameters and Seeds

For all training logs of all models trained in the paper see `documentation/training_logs/..`.
These fills contain printed all hyperparameters, as well as the training and development performance over time and the seeds used in training.

## Demo: training a model on a dummy dataset
You can try this demo on almost any machine, it requires no GPU and runs in a couple of seconds on a laptop with a CPU containing 8 GB RAM.

### Loading the data
To get a feel of the data used in the paper, we can look at the data in the folder `data/demo_dataset/..`. This dataset is highly simplified in terms of grid size, vocabulary, and number of examples, but the ideas are the same. When opening `data/demo_dataset/dataset.txt` we can see that the first example if we follow the keys "examples" and "situational_1" is the following: 

<details open>
<summary>The first data example in the split called "situational_1" (i.e., novel direction) set. Click to open/close.</summary>
<p>
 
```javascript
{
                "command": "walk,to,a,red,circle",
                "meaning": "walk,to,a,red,circle",
                "derivation": "NP -> NN,NP -> JJ NP,DP -> 'a' NP,VP -> VV_intrans 'to' DP,ROOT -> VP;T:walk,NT:VV_intransitive -> walk,T:to,T:a,T:red,NT:JJ -> red,T:circle,NT:NN -> circle",
                "situation": {
                    "grid_size": 4,
                    "agent_position": {
                        "row": "2",
                        "column": "3"
                    },
                    "agent_direction": 0,
                    "target_object": {
                        "vector": "1000101000",
                        "position": {
                            "row": "3",
                            "column": "2"
                        },
                        "object": {
                            "shape": "circle",
                            "color": "red",
                            "size": "1"
                        }
                    },
                    "distance_to_target": "2",
                    "direction_to_target": "sw",
                    "placed_objects": {
                        "0": {
                            "vector": "1000101000",
                            "position": {
                                "row": "3",
                                "column": "2"
                            },
                            "object": {
                                "shape": "circle",
                                "color": "red",
                                "size": "1"
                            }
                        },
                        "1": {
                            "vector": "0010011000",
                            "position": {
                                "row": "2",
                                "column": "1"
                            },
                            "object": {
                                "shape": "square",
                                "color": "red",
                                "size": "3"
                            }
                        },
                        "2": {
                            "vector": "0100010001",
                            "position": {
                                "row": "1",
                                "column": "2"
                            },
                            "object": {
                                "shape": "square",
                                "color": "blue",
                                "size": "2"
                            }
                        },
                        "3": {
                            "vector": "0001010100",
                            "position": {
                                "row": "1",
                                "column": "1"
                            },
                            "object": {
                                "shape": "square",
                                "color": "green",
                                "size": "4"
                            }
                        }
                    },
                    "carrying_object": null
                },
                "target_commands": "turn left,turn left,walk,turn left,walk",
                "verb_in_command": "walk",
                "manner": "",
                "referred_target": " red circle"
            }
```

</p>
</details>

This data example contains the *"command"*, or input  instruction, 'walk to the red circle', that in this case based on the situation maps to the target command sequence of *"target_commands"*: "turn left,turn left,walk,turn left,walk". The data example contains the situation representation, or world state, at the key *"situation"*, and it also contains some additional information that is useful in parsing it back to the representation it was generated from, namely the *"derivation"* containing the depth-first extracted constituency tree, the *"meaning"* containing the semantic meaning of the input instruction. This is only useful if we would have generated the benchmark with nonsensical words, in that case we would need a semantic representation that can be parsed by humans. 

This example is visualized by the following animation:

![demo_example](https://raw.githubusercontent.com/LauraRuis/multimodal_seq2seq_gSCAN/master/data/demo_dataset/walk_to_a_red_circle/situation_1/movie.gif)

For a way to parse the dataset independently of the code in `GroundedScan`, refer to the folder `read_gscan` which has its own `readme.md` with a demonstration. 

We can train the baseline model from the paper on this demo dataset with the following command:


```>> python3.7 -m seq2seq --mode=train --data_directory=data/demo_dataset --embedding_dimension=5 --encoder_hidden_size=20 --decoder_hidden_size=20 --max_training_iterations=1000 --training_batch_size=5 --print_every=100 --evaluate_every=500 --generate_vocabularies```

This will parse the file `data/demo_dataset/dataset.txt` and extract the trainnig set and development set for training, and convert the to a representation that can be parsed by numerical methods. The situation representation is parsed by the code in `seq2seq/gSCAN_dataset.py`. For the example shown in the previous subsection, the world state will be represented by a 4 x 4 x 15 sized matrix of the grid world, where the 15 dimensions are the following:

[size 1, size 2, size 3, size 4, circle, square, red, green, yellow, blue, agent, east, south, west, north]

The first vector representing one grid cell, namely the green square of size 4 in row 1 and column 1 (starting at 0) of the world, will be the following vector: `[0, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0]`.

### Training a model

The command mentioned earlier for training, will generate the following output (containing all set parameters, and logging the training progress in terms of accuracy, loss, and exact match. The evaluation cycles are done on the development set.): 

<details open>
<summary>The training command will produce the following output (click to open/close): </summary>
<p>
 
```python
2020-03-10 19:53 mode: train
2020-03-10 19:53 output_directory: output
2020-03-10 19:53 resume_from_file: 
2020-03-10 19:53 split: test
2020-03-10 19:53 data_directory: data/demo_dataset
2020-03-10 19:53 input_vocab_path: training_input_vocab.txt
2020-03-10 19:53 target_vocab_path: training_target_vocab.txt
2020-03-10 19:53 generate_vocabularies: True
2020-03-10 19:53 training_batch_size: 5
2020-03-10 19:53 k: 0
2020-03-10 19:53 test_batch_size: 1
2020-03-10 19:53 max_training_examples: None
2020-03-10 19:53 learning_rate: 0.001
2020-03-10 19:53 lr_decay: 0.9
2020-03-10 19:53 lr_decay_steps: 20000
2020-03-10 19:53 adam_beta_1: 0.9
2020-03-10 19:53 adam_beta_2: 0.999
2020-03-10 19:53 print_every: 100
2020-03-10 19:53 evaluate_every: 500
2020-03-10 19:53 max_training_iterations: 1000
2020-03-10 19:53 weight_target_loss: 0.3
2020-03-10 19:53 max_testing_examples: None
2020-03-10 19:53 splits: test
2020-03-10 19:53 max_decoding_steps: 30
2020-03-10 19:53 output_file_name: predict.json
2020-03-10 19:53 simple_situation_representation: True
2020-03-10 19:53 cnn_hidden_num_channels: 50
2020-03-10 19:53 cnn_kernel_size: 7
2020-03-10 19:53 cnn_dropout_p: 0.1
2020-03-10 19:53 auxiliary_task: False
2020-03-10 19:53 embedding_dimension: 5
2020-03-10 19:53 num_encoder_layers: 1
2020-03-10 19:53 encoder_hidden_size: 20
2020-03-10 19:53 encoder_dropout_p: 0.3
2020-03-10 19:53 encoder_bidirectional: True
2020-03-10 19:53 num_decoder_layers: 1
2020-03-10 19:53 attention_type: bahdanau
2020-03-10 19:53 decoder_dropout_p: 0.3
2020-03-10 19:53 decoder_hidden_size: 20
2020-03-10 19:53 conditional_attention: True
2020-03-10 19:53 seed: 42
2020-03-10 19:53 Loading Training set...
2020-03-10 19:53 Verb-adverb combinations in training set: 
2020-03-10 19:53 Verbs for adverb: 
2020-03-10 19:53    walk: 1531 occurrences.
2020-03-10 19:53 Verb-adverb combinations in dev set: 
2020-03-10 19:53 Verbs for adverb: 
2020-03-10 19:53    walk: 170 occurrences.
2020-03-10 19:53 Generating vocabularies...
2020-03-10 19:53 Populating vocabulary...
2020-03-10 19:53 Done generating vocabularies.
2020-03-10 19:53 Converting dataset to tensors...
2020-03-10 19:53 Done Loading Training set.
2020-03-10 19:53   Loaded 1531 training examples.
2020-03-10 19:53   Input vocabulary size training set: 14
2020-03-10 19:53   Most common input words: [('walk', 1531), ('to', 1531), ('a', 1531), ('circle', 869), ('square', 662)]
2020-03-10 19:53   Output vocabulary size training set: 6
2020-03-10 19:53   Most common target words: [('walk', 4794), ('turn left', 1361), ('turn right', 755)]
2020-03-10 19:53 Saved vocabularies to training_input_vocab.txt for input and training_target_vocab.txt for target.
2020-03-10 19:53 Loading Dev. set...
2020-03-10 19:53 Verb-adverb combinations in training set: 
2020-03-10 19:53 Verbs for adverb: 
2020-03-10 19:53    walk: 1531 occurrences.
2020-03-10 19:53 Verb-adverb combinations in dev set: 
2020-03-10 19:53 Verbs for adverb: 
2020-03-10 19:53    walk: 170 occurrences.
2020-03-10 19:53 Loading vocabularies...
2020-03-10 19:53 Done loading vocabularies.
2020-03-10 19:53 Converting dataset to tensors...
2020-03-10 19:53 Done Loading Dev. set.
2020-03-10 19:53 Total parameters: 74670
2020-03-10 19:53 situation_encoder.conv_1.weight : [50, 15, 1, 1]
2020-03-10 19:53 situation_encoder.conv_1.bias : [50]
2020-03-10 19:53 situation_encoder.conv_2.weight : [50, 15, 5, 5]
2020-03-10 19:53 situation_encoder.conv_2.bias : [50]
2020-03-10 19:53 situation_encoder.conv_3.weight : [50, 15, 7, 7]
2020-03-10 19:53 situation_encoder.conv_3.bias : [50]
2020-03-10 19:53 visual_attention.key_layer.weight : [20, 150]
2020-03-10 19:53 visual_attention.query_layer.weight : [20, 20]
2020-03-10 19:53 visual_attention.energy_layer.weight : [1, 20]
2020-03-10 19:53 encoder.embedding.weight : [14, 5]
2020-03-10 19:53 encoder.lstm.weight_ih_l0 : [80, 5]
2020-03-10 19:53 encoder.lstm.weight_hh_l0 : [80, 20]
2020-03-10 19:53 encoder.lstm.bias_ih_l0 : [80]
2020-03-10 19:53 encoder.lstm.bias_hh_l0 : [80]
2020-03-10 19:53 encoder.lstm.weight_ih_l0_reverse : [80, 5]
2020-03-10 19:53 encoder.lstm.weight_hh_l0_reverse : [80, 20]
2020-03-10 19:53 encoder.lstm.bias_ih_l0_reverse : [80]
2020-03-10 19:53 encoder.lstm.bias_hh_l0_reverse : [80]
2020-03-10 19:53 enc_hidden_to_dec_hidden.weight : [20, 20]
2020-03-10 19:53 enc_hidden_to_dec_hidden.bias : [20]
2020-03-10 19:53 textual_attention.key_layer.weight : [20, 20]
2020-03-10 19:53 textual_attention.query_layer.weight : [20, 20]
2020-03-10 19:53 textual_attention.energy_layer.weight : [1, 20]
2020-03-10 19:53 attention_decoder.queries_to_keys.weight : [20, 40]
2020-03-10 19:53 attention_decoder.queries_to_keys.bias : [20]
2020-03-10 19:53 attention_decoder.embedding.weight : [6, 20]
2020-03-10 19:53 attention_decoder.lstm.weight_ih_l0 : [80, 60]
2020-03-10 19:53 attention_decoder.lstm.weight_hh_l0 : [80, 20]
2020-03-10 19:53 attention_decoder.lstm.bias_ih_l0 : [80]
2020-03-10 19:53 attention_decoder.lstm.bias_hh_l0 : [80]
2020-03-10 19:53 attention_decoder.output_to_hidden.weight : [20, 80]
2020-03-10 19:53 attention_decoder.hidden_to_output.weight : [6, 20]
2020-03-10 19:53 Training starts..
2020-03-10 19:53 Iteration 00000100, loss   0.9701, accuracy 51.72, exact match  0.00, learning_rate 0.00100, aux. accuracy target pos  0.00
2020-03-10 19:53 Iteration 00000200, loss   0.9381, accuracy 47.62, exact match  0.00, learning_rate 0.00100, aux. accuracy target pos  0.00
2020-03-10 19:53 Iteration 00000300, loss   0.7489, accuracy 78.26, exact match 20.00, learning_rate 0.00100, aux. accuracy target pos  0.00
2020-03-10 19:53 Iteration 00000400, loss   0.6548, accuracy 74.19, exact match  0.00, learning_rate 0.00100, aux. accuracy target pos  0.00
2020-03-10 19:53 Iteration 00000500, loss   0.5565, accuracy 81.25, exact match 20.00, learning_rate 0.00100, aux. accuracy target pos  0.00
2020-03-10 19:53 Evaluating..
2020-03-10 19:53 Predicted for 170 examples.
2020-03-10 19:53 Done predicting in 1.1095800399780273 seconds.
2020-03-10 19:53   Evaluation Accuracy: 52.94 Exact Match:  5.88  Target Accuracy:  0.00
2020-03-10 19:53 Iteration 00000600, loss   0.5894, accuracy 79.17, exact match 20.00, learning_rate 0.00100, aux. accuracy target pos  0.00
2020-03-10 19:53 Iteration 00000700, loss   0.5319, accuracy 77.78, exact match 20.00, learning_rate 0.00100, aux. accuracy target pos  0.00
2020-03-10 19:53 Iteration 00000800, loss   0.3924, accuracy 83.33, exact match 20.00, learning_rate 0.00100, aux. accuracy target pos  0.00
2020-03-10 19:53 Iteration 00000900, loss   0.5259, accuracy 81.48, exact match 20.00, learning_rate 0.00100, aux. accuracy target pos  0.00
2020-03-10 19:53 Iteration 00001000, loss   0.3916, accuracy 85.29, exact match 20.00, learning_rate 0.00099, aux. accuracy target pos  0.00
2020-03-10 19:53 Evaluating..
2020-03-10 19:53 Predicted for 170 examples.
2020-03-10 19:53 Done predicting in 1.1327424049377441 seconds.
2020-03-10 19:53   Evaluation Accuracy: 58.40 Exact Match: 20.59  Target Accuracy:  0.00
2020-03-10 19:53 Finished training.

```

</p>
</details>


## Testing the saved model

The training command has saved the best model in `output/model_best.pth.tar` (the directory specified by `--output_directory`). We can now use the model for predictions with the following command (lets do predictions for the test set, the visual split (i.e., 'red squares'), and the situational_1 split (i.e., 'novel direction')): 

```python3.7 -m seq2seq --mode=test --data_directory=data/demo_dataset --embedding_dimension=5 --encoder_hidden_size=20 --decoder_hidden_size=20 --resume_from_file=output/model_best.pth.tar --splits=test,visual,situational_1 --max_decoding_steps=50```

<details open>
<summary>This produces the followig output: (click to open/close): </summary>
<p>
 
```python
2020-03-10 20:00 mode: test
2020-03-10 20:00 output_directory: output
2020-03-10 20:00 resume_from_file: output/model_best.pth.tar
2020-03-10 20:00 split: test
2020-03-10 20:00 data_directory: data/demo_dataset
2020-03-10 20:00 input_vocab_path: training_input_vocab.txt
2020-03-10 20:00 target_vocab_path: training_target_vocab.txt
2020-03-10 20:00 generate_vocabularies: False
2020-03-10 20:00 training_batch_size: 50
2020-03-10 20:00 k: 0
2020-03-10 20:00 test_batch_size: 1
2020-03-10 20:00 max_training_examples: None
2020-03-10 20:00 learning_rate: 0.001
2020-03-10 20:00 lr_decay: 0.9
2020-03-10 20:00 lr_decay_steps: 20000
2020-03-10 20:00 adam_beta_1: 0.9
2020-03-10 20:00 adam_beta_2: 0.999
2020-03-10 20:00 print_every: 100
2020-03-10 20:00 evaluate_every: 1000
2020-03-10 20:00 max_training_iterations: 100000
2020-03-10 20:00 weight_target_loss: 0.3
2020-03-10 20:00 max_testing_examples: None
2020-03-10 20:00 splits: test,visual,situational_1
2020-03-10 20:00 max_decoding_steps: 50
2020-03-10 20:00 output_file_name: predict.json
2020-03-10 20:00 simple_situation_representation: True
2020-03-10 20:00 cnn_hidden_num_channels: 50
2020-03-10 20:00 cnn_kernel_size: 7
2020-03-10 20:00 cnn_dropout_p: 0.1
2020-03-10 20:00 auxiliary_task: False
2020-03-10 20:00 embedding_dimension: 5
2020-03-10 20:00 num_encoder_layers: 1
2020-03-10 20:00 encoder_hidden_size: 20
2020-03-10 20:00 encoder_dropout_p: 0.3
2020-03-10 20:00 encoder_bidirectional: True
2020-03-10 20:00 num_decoder_layers: 1
2020-03-10 20:00 attention_type: bahdanau
2020-03-10 20:00 decoder_dropout_p: 0.3
2020-03-10 20:00 decoder_hidden_size: 20
2020-03-10 20:00 conditional_attention: True
2020-03-10 20:00 seed: 42
2020-03-10 20:00 Loading test dataset split...
2020-03-10 20:00 Verb-adverb combinations in training set: 
2020-03-10 20:00 Verbs for adverb: 
2020-03-10 20:00    walk: 1531 occurrences.
2020-03-10 20:00 Verb-adverb combinations in dev set: 
2020-03-10 20:00 Verbs for adverb: 
2020-03-10 20:00    walk: 170 occurrences.
2020-03-10 20:00 Loading vocabularies...
2020-03-10 20:00 Done loading vocabularies.
2020-03-10 20:00 Converting dataset to tensors...
2020-03-10 20:00 Done Loading test dataset split.
2020-03-10 20:00   Loaded 305 examples.
2020-03-10 20:00   Input vocabulary size: 14
2020-03-10 20:00   Most common input words: [('walk', 1531), ('to', 1531), ('a', 1531), ('circle', 869), ('square', 662)]
2020-03-10 20:00   Output vocabulary size: 6
2020-03-10 20:00   Most common target words: [('walk', 4794), ('turn left', 1361), ('turn right', 755)]
2020-03-10 20:00 Loading checkpoint from file at 'output/model_best.pth.tar'
2020-03-10 20:00 Loaded checkpoint 'output/model_best.pth.tar' (iter 1002)
2020-03-10 20:00 Predicted for 305 examples.
2020-03-10 20:00 Done predicting in 2.7643723487854004 seconds.
2020-03-10 20:00 Wrote predictions for 305 examples.
2020-03-10 20:00 Saved predictions to output/test_predict.json
2020-03-10 20:00 Loading visual dataset split...
2020-03-10 20:00 Verb-adverb combinations in training set: 
2020-03-10 20:00 Verbs for adverb: 
2020-03-10 20:00    walk: 1531 occurrences.
2020-03-10 20:00 Verb-adverb combinations in dev set: 
2020-03-10 20:00 Verbs for adverb: 
2020-03-10 20:00    walk: 170 occurrences.
2020-03-10 20:00 Loading vocabularies...
2020-03-10 20:00 Done loading vocabularies.
2020-03-10 20:00 Converting dataset to tensors...
2020-03-10 20:00 Done Loading test dataset split.
2020-03-10 20:00   Loaded 378 examples.
2020-03-10 20:00   Input vocabulary size: 14
2020-03-10 20:00   Most common input words: [('walk', 1531), ('to', 1531), ('a', 1531), ('circle', 869), ('square', 662)]
2020-03-10 20:00   Output vocabulary size: 6
2020-03-10 20:00   Most common target words: [('walk', 4794), ('turn left', 1361), ('turn right', 755)]
2020-03-10 20:00 Loading checkpoint from file at 'output/model_best.pth.tar'
2020-03-10 20:00 Loaded checkpoint 'output/model_best.pth.tar' (iter 1002)
2020-03-10 20:00 Predicted for 378 examples.
2020-03-10 20:00 Done predicting in 2.68125581741333 seconds.
2020-03-10 20:00 Wrote predictions for 378 examples.
2020-03-10 20:00 Saved predictions to output/visual_predict.json
2020-03-10 20:00 Loading situational_1 dataset split...
2020-03-10 20:00 Verb-adverb combinations in training set: 
2020-03-10 20:00 Verbs for adverb: 
2020-03-10 20:00    walk: 1531 occurrences.
2020-03-10 20:00 Verb-adverb combinations in dev set: 
2020-03-10 20:00 Verbs for adverb: 
2020-03-10 20:00    walk: 170 occurrences.
2020-03-10 20:00 Loading vocabularies...
2020-03-10 20:00 Done loading vocabularies.
2020-03-10 20:00 Converting dataset to tensors...
2020-03-10 20:00 Done Loading test dataset split.
2020-03-10 20:00   Loaded 450 examples.
2020-03-10 20:00   Input vocabulary size: 14
2020-03-10 20:00   Most common input words: [('walk', 1531), ('to', 1531), ('a', 1531), ('circle', 869), ('square', 662)]
2020-03-10 20:00   Output vocabulary size: 6
2020-03-10 20:00   Most common target words: [('walk', 4794), ('turn left', 1361), ('turn right', 755)]
2020-03-10 20:00 Loading checkpoint from file at 'output/model_best.pth.tar'
2020-03-10 20:00 Loaded checkpoint 'output/model_best.pth.tar' (iter 1002)
2020-03-10 20:00 Predicted for 450 examples.
2020-03-10 20:00 Done predicting in 3.259852170944214 seconds.
2020-03-10 20:00 Wrote predictions for 450 examples.
2020-03-10 20:00 Saved predictions to output/situational_1_predict.json

```

</p>
</details>

Now the folder specified with `--output_directory` `output` contains the predictions in the files `test_predict.json`, `visual_predict.json`, and `situational_1_predict.json`. 
 
 <details open>
<summary>For example, the first prediction of `situational_1_predict.json`
 when this example was run is the following: (click to open/close): </summary>
<p>
 
```javascript
 {
        "input": [
            "walk",
            "to",
            "a",
            "red",
            "circle"
        ],
        "prediction": [
            "turn left",
            "turn left",
            "walk",
            "turn right",
            "walk"
        ],
        "derivation": [
            "NP -> NN,NP -> JJ NP,DP -> 'a' NP,VP -> VV_intrans 'to' DP,ROOT -> VP;T:walk,NT:VV_intransitive -> walk,T:to,T:a,T:red,NT:JJ -> red,T:circle,NT:NN -> circle"
        ],
        "target": [
            "turn left",
            "turn left",
            "walk",
            "turn left",
            "walk"
        ],
        "situation": [
            {
                "grid_size": 4,
                "agent_position": {
                    "row": "2",
                    "column": "3"
                },
                "agent_direction": 0,
                "target_object": {
                    "vector": "1000101000",
                    "position": {
                        "row": "3",
                        "column": "2"
                    },
                    "object": {
                        "shape": "circle",
                        "color": "red",
                        "size": "1"
                    }
                },
                "distance_to_target": "2",
                "direction_to_target": "sw",
                "placed_objects": {
                    "0": {
                        "vector": "1000101000",
                        "position": {
                            "row": "3",
                            "column": "2"
                        },
                        "object": {
                            "shape": "circle",
                            "color": "red",
                            "size": "1"
                        }
                    },
                    "1": {
                        "vector": "0010011000",
                        "position": {
                            "row": "2",
                            "column": "1"
                        },
                        "object": {
                            "shape": "square",
                            "color": "red",
                            "size": "3"
                        }
                    },
                    "2": {
                        "vector": "0100010001",
                        "position": {
                            "row": "1",
                            "column": "2"
                        },
                        "object": {
                            "shape": "square",
                            "color": "blue",
                            "size": "2"
                        }
                    },
                    "3": {
                        "vector": "0001010100",
                        "position": {
                            "row": "1",
                            "column": "1"
                        },
                        "object": {
                            "shape": "square",
                            "color": "green",
                            "size": "4"
                        }
                    }
                },
                "carrying_object": null
            }
        ],
        "attention_weights_input": [
            [
                [
                    0.1347028911113739,
                    0.13788217306137085,
                    0.1408216804265976,
                    0.14514818787574768,
                    0.1460469663143158,
                    0.1525878757238388,
                    0.14281028509140015
                ]
            ],
            [
                [
                    0.13317446410655975,
                    0.13619999587535858,
                    0.13789284229278564,
                    0.1448119431734085,
                    0.14822974801063538,
                    0.15437810122966766,
                    0.14531295001506805
                ]
            ],
            [
                [
                    0.12259363383054733,
                    0.130519300699234,
                    0.13570959866046906,
                    0.14702339470386505,
                    0.1503848433494568,
                    0.1618456244468689,
                    0.15192364156246185
                ]
            ],
            [
                [
                    0.1116199865937233,
                    0.12276129424571991,
                    0.13262240588665009,
                    0.14890818297863007,
                    0.15098372101783752,
                    0.17331182956695557,
                    0.15979261696338654
                ]
            ],
            [
                [
                    0.12063928693532944,
                    0.12789921462535858,
                    0.13446803390979767,
                    0.1472683697938919,
                    0.15150536596775055,
                    0.1624067723751068,
                    0.15581294894218445
                ]
            ]
        ],
        "attention_weights_situation": [
            [
                [
                    0.03381653502583504,
                    0.05744897574186325,
                    0.07042934745550156,
                    0.055202607065439224,
                    0.028679050505161285,
                    0.04473859816789627,
                    0.055586643517017365,
                    0.05163794383406639,
                    0.025743210688233376,
                    0.05598282441496849,
                    0.11211208254098892,
                    0.10360352694988251,
                    0.030320605263113976,
                    0.09432610124349594,
                    0.13378305733203888,
                    0.04658888652920723
                ]
            ],
            [
                [
                    0.030180441215634346,
                    0.054645419120788574,
                    0.06971719115972519,
                    0.05268885940313339,
                    0.025461191311478615,
                    0.042079515755176544,
                    0.0535975806415081,
                    0.05010606721043587,
                    0.02283935621380806,
                    0.05391918495297432,
                    0.11132807284593582,
                    0.11724720150232315,
                    0.030814023688435555,
                    0.10024137794971466,
                    0.14098992943763733,
                    0.04414454475045204
                ]
            ],
            [
                [
                    0.02856718935072422,
                    0.04643532261252403,
                    0.06758377701044083,
                    0.0482352077960968,
                    0.02746320702135563,
                    0.03763550519943237,
                    0.04657653719186783,
                    0.04258711263537407,
                    0.026288190856575966,
                    0.043896254152059555,
                    0.10679258406162262,
                    0.12164262682199478,
                    0.04711860045790672,
                    0.11570236086845398,
                    0.1548127979040146,
                    0.03866272047162056
                ]
            ],
            [
                [
                    0.02986297383904457,
                    0.04268667474389076,
                    0.06872523576021194,
                    0.048437826335430145,
                    0.030498236417770386,
                    0.037579216063022614,
                    0.04487932100892067,
                    0.041791316121816635,
                    0.030200282111763954,
                    0.04008848965167999,
                    0.09280946850776672,
                    0.13625997304916382,
                    0.057449717074632645,
                    0.12127335369586945,
                    0.13907437026500702,
                    0.03838350996375084
                ]
            ],
            [
                [
                    0.022747689858078957,
                    0.03625047951936722,
                    0.06052326411008835,
                    0.045950260013341904,
                    0.026497578248381615,
                    0.035583287477493286,
                    0.045341309159994125,
                    0.04634739086031914,
                    0.028446927666664124,
                    0.037730518728494644,
                    0.08468840271234512,
                    0.18837516009807587,
                    0.08500255644321442,
                    0.10905778408050537,
                    0.11035732179880142,
                    0.03710004314780235
                ]
            ]
        ],
        "accuracy": 80.0,
        "exact_match": false,
        "position_accuracy": 0
    }
 ```

</p>
</details>

These predictions also contain the attentio weights for the attention over the commands as well as the attention over the world state. The .json files with the predictions can be passed to the modes `error_analysis`, and `execute_commands` in the [groundedSCAN repository](https://github.com/LauraRuis/groundedSCAN). If we place `situational_1_predict.json` in the folder specified by `--output_directory` of that dataset (for instructions see the README in the groundedSCAN repo under Using the repository -- Execute commands) and we run the following command with that repository:

```>> python3.7 -m GroundedScan --mode=execute_commands --load_dataset_from=data/demo_dataset/dataset.txt --predicted_commands_file=situational_1_predict.json```

We created the animated visualizations with attention, like the one for our running example: 

![predicted_ex](https://raw.githubusercontent.com/LauraRuis/multimodal_seq2seq_gSCAN/master/documentation/prediction_example.gif)

Which is not exactly correct, but we didn't train for that long :). The darker the grid cell, the higher the attention to that cell. In this example the attention doesn't see to know exactly what to do, but for converged models with correct examples the attention is usually dark on the target object.

