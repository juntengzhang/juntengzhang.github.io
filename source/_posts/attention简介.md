---
title: attention简介
date: 2019-04-25 16:45:48
tags:
---


有关attention的源码在'lib/python3.5/site-packages/tensorflow/contrib/seq2seq'，一个attention解码器例子：

<!--more-->

```
def attention_decoder(inputs, memory, num_units=None, scope="attention_decoder", reuse=None):
    '''Applies a GRU to `inputs`, while attending `memory`.
    Args:
      inputs: A 3d tensor with shape of [N, T', C']. Decoder inputs.
      memory: A 3d tensor with shape of [N, T, C]. Outputs of encoder network.
      num_units: An int. Attention size.
      scope: Optional scope for `variable_scope`.
      reuse: Boolean, whether to reuse the weights of a previous layer
        by the same name.

    Returns:
      A 3d tensor with shape of [N, T, num_units].
    '''
    with tf.variable_scope(scope, reuse=reuse):
        if num_units is None:
            num_units = inputs.get_shape().as_list[-1]
        #attention机制，根据memory和query生成alignm,ents
        attention_mechanism = tf.contrib.seq2seq.BahdanauAttention(num_units,
                                                                   memory)
        #标准GRU单元，继承RNN
        decoder_cell = tf.contrib.rnn.GRUCell(num_units)
        #attention单元，继承RNN
        cell_with_attention = tf.contrib.seq2seq.AttentionWrapper(decoder_cell,
                                                                  attention_mechanism,
                                                                  num_units,
                                                                  alignment_history=True)
        #动态rnn
        outputs, state = tf.nn.dynamic_rnn(cell_with_attention, inputs, dtype=tf.float32) #( N, T', 16)

    return outputs, state
```
attention_mechanism根据memory和query生成alignments；decoder_cell为rnn单元；AttentionWrapper封装decoder_cell和attention_mechanism生成新的attention单元；下面详细讲解。

## attention_mechanism
```
 def _bahdanau_score(processed_query, keys, normalize):
   #根据query和memory(key)生成score，作为softmax的输入
   """Implements Bahdanau-style (additive) scoring function.

   This attention has two forms.  The first is Bhandanau attention,
   as described in:

   Dzmitry Bahdanau, Kyunghyun Cho, Yoshua Bengio.
   "Neural Machine Translation by Jointly Learning to Align and Translate."
   ICLR 2015. https://arxiv.org/abs/1409.0473

   The second is the normalized form.  This form is inspired by the
   weight normalization article:

   Tim Salimans, Diederik P. Kingma.
   "Weight Normalization: A Simple Reparameterization to Accelerate
    Training of Deep Neural Networks."
   https://arxiv.org/abs/1602.07868

   To enable the second form, set `normalize=True`.

   Args:
     processed_query: Tensor, shape `[batch_size, num_units]` to compare to keys.
     keys: Processed memory, shape `[batch_size, max_time, num_units]`.
     normalize: Whether to normalize the score function.

   Returns:
     A `[batch_size, max_time]` tensor of unnormalized score values.
   """
   dtype = processed_query.dtype
   # Get the number of hidden units from the trailing dimension of keys
   num_units = keys.shape[2].value or array_ops.shape(keys)[2]
   # Reshape from [batch_size, ...] to [batch_size, 1, ...] for broadcasting.
   processed_query = array_ops.expand_dims(processed_query, 1)
   v = variable_scope.get_variable(
       "attention_v", [num_units], dtype=dtype)
   if normalize:
     # Scalar used in weight normalization
     g = variable_scope.get_variable(
         "attention_g", dtype=dtype,
         initializer=init_ops.constant_initializer(math.sqrt((1. / num_units))),
         shape=())
     # Bias added prior to the nonlinearity
     b = variable_scope.get_variable(
         "attention_b", [num_units], dtype=dtype,
         initializer=init_ops.zeros_initializer())
     # normed_v = g * v / ||v||
     normed_v = g * v * math_ops.rsqrt(
         math_ops.reduce_sum(math_ops.square(v)))
     return math_ops.reduce_sum(
         normed_v * math_ops.tanh(keys + processed_query + b), [2])
   else:
     return math_ops.reduce_sum(v * math_ops.tanh(keys + processed_query), [2])

 class BahdanauAttention(_BaseAttentionMechanism):
   """Implements Bahdanau-style (additive) attention.

   This attention has two forms.  The first is Bahdanau attention,
   as described in:

   Dzmitry Bahdanau, Kyunghyun Cho, Yoshua Bengio.
   "Neural Machine Translation by Jointly Learning to Align and Translate."
   ICLR 2015. https://arxiv.org/abs/1409.0473

   The second is the normalized form.  This form is inspired by the
   weight normalization article:

   Tim Salimans, Diederik P. Kingma.
   "Weight Normalization: A Simple Reparameterization to Accelerate
    Training of Deep Neural Networks."
   https://arxiv.org/abs/1602.07868

   To enable the second form, construct the object with parameter
   `normalize=True`.
   """
   def __init__(self,
                num_units,
                memory,
                memory_sequence_length=None,
                normalize=False,
                probability_fn=None,
                score_mask_value=None,
                dtype=None,
                name="BahdanauAttention"):
     """Construct the Attention mechanism.

     Args:
       num_units: The depth of the query mechanism.
       memory: The memory to query; usually the output of an RNN encoder.  This
         tensor should be shaped `[batch_size, max_time, ...]`.
       memory_sequence_length (optional): Sequence lengths for the batch entries
         in memory.  If provided, the memory tensor rows are masked with zeros
         for values past the respective sequence lengths.
       normalize: Python boolean.  Whether to normalize the energy term.
       probability_fn: (optional) A `callable`.  Converts the score to
         probabilities.  The default is `tf.nn.softmax`. Other options include
         `tf.contrib.seq2seq.hardmax` and `tf.contrib.sparsemax.sparsemax`.
         Its signature should be: `probabilities = probability_fn(score)`.
       score_mask_value: (optional): The mask value for score before passing into
         `probability_fn`. The default is -inf. Only used if
         `memory_sequence_length` is not None.
       dtype: The data type for the query and memory layers of the attention
         mechanism.
       name: Name to use when creating ops.
     """
     if probability_fn is None:
       probability_fn = nn_ops.softmax
     if dtype is None:
       dtype = dtypes.float32
     wrapped_probability_fn = lambda score, _: probability_fn(score)
     super(BahdanauAttention, self).__init__(
         query_layer=layers_core.Dense(
             num_units, name="query_layer", use_bias=False, dtype=dtype),
         memory_layer=layers_core.Dense(
             num_units, name="memory_layer", use_bias=False, dtype=dtype),
         memory=memory,
         probability_fn=wrapped_probability_fn,
         memory_sequence_length=memory_sequence_length,
         score_mask_value=score_mask_value,
         name=name)
     self._num_units = num_units
     self._normalize = normalize
     self._name = name

   def __call__(self, query, state):
     """Score the query based on the keys and values.

     Args:
       query: Tensor of dtype matching `self.values` and shape
         `[batch_size, query_depth]`.
       state: Tensor of dtype matching `self.values` and shape
         `[batch_size, alignments_size]`
         (`alignments_size` is memory's `max_time`).
         pre step的输出，即score经过softmax，参考next_state

     Returns:
       alignments: Tensor of dtype matching `self.values` and shape
         `[batch_size, alignments_size]` (`alignments_size` is memory's
         `max_time`).
     """
     with variable_scope.variable_scope(None, "bahdanau_attention", [query]):
       processed_query = self.query_layer(query) if self.query_layer else query
       score = _bahdanau_score(processed_query, self._keys, self._normalize)
     alignments = self._probability_fn(score, state)
     next_state = alignments
     return alignments, next_state
```

## AttentionWrapper
```
 def _compute_attention(attention_mechanism, cell_output, attention_state,
                        attention_layer):
   """Computes the attention and alignments for a given attention_mechanism."""
   alignments, next_attention_state = attention_mechanism(
       cell_output, state=attention_state)

   # Reshape from [batch_size, memory_time] to [batch_size, 1, memory_time]
   expanded_alignments = array_ops.expand_dims(alignments, 1)
   # Context is the inner product of alignments and values along the
   # memory time dimension.
   # alignments shape is
   #   [batch_size, 1, memory_time]
   # attention_mechanism.values shape is
   #   [batch_size, memory_time, memory_size]
   # the batched matmul is over memory_time, so the output shape is
   #   [batch_size, 1, memory_size].
   # we then squeeze out the singleton dim.
   context = math_ops.matmul(expanded_alignments, attention_mechanism.values)
   context = array_ops.squeeze(context, [1])

   if attention_layer is not None:
     attention = attention_layer(array_ops.concat([cell_output, context], 1))
   else:
     attention = context

   return attention, alignments, next_attention_state

 class AttentionWrapperState(
     collections.namedtuple("AttentionWrapperState",
                            ("cell_state", "attention", "time", "alignments",
                             "alignment_history", "attention_state"))):
   """`namedtuple` storing the state of a `AttentionWrapper`.

   Contains:

     - `cell_state`: The state of the wrapped `RNNCell` at the previous time
       step.
     - `attention`: The attention emitted at the previous time step.
     - `time`: int32 scalar containing the current time step.
     - `alignments`: A single or tuple of `Tensor`(s) containing the alignments
        emitted at the previous time step for each attention mechanism.
     - `alignment_history`: (if enabled) a single or tuple of `TensorArray`(s)
        containing alignment matrices from all time steps for each attention
        mechanism. Call `stack()` on each to convert to a `Tensor`.
     - `attention_state`: A single or tuple of nested objects
        containing attention mechanism state for each attention mechanism.
        The objects may contain Tensors or TensorArrays.
   """

 class AttentionWrapper(rnn_cell_impl.RNNCell):
   """Wraps another `RNNCell` with attention.
   """

   def __init__(self,
                cell,
                attention_mechanism,
                attention_layer_size=None,
                alignment_history=False,
                cell_input_fn=None,
                output_attention=True,
                initial_cell_state=None,
                name=None,
                attention_layer=None):
     """Construct the `AttentionWrapper`.
     Args:
       cell: An instance of `RNNCell`.
       attention_mechanism: A list of `AttentionMechanism` instances or a single
         instance.
       attention_layer_size: A list of Python integers or a single Python
         integer, the depth of the attention (output) layer(s). If None
         (default), use the context as attention at each time step. Otherwise,
         feed the context and cell output into the attention layer to generate
         attention at each time step. If attention_mechanism is a list,
         attention_layer_size must be a list of the same length. If
         attention_layer is set, this must be None.
       alignment_history: Python boolean, whether to store alignment history
         from all time steps in the final output state (currently stored as a
         time major `TensorArray` on which you must call `stack()`).
       cell_input_fn: (optional) A `callable`.  The default is:
         `lambda inputs, attention: array_ops.concat([inputs, attention], -1)`.
       output_attention: Python bool.  If `True` (default), the output at each
         time step is the attention value.  This is the behavior of Luong-style
         attention mechanisms.  If `False`, the output at each time step is
         the output of `cell`.  This is the behavior of Bhadanau-style
         attention mechanisms.  In both cases, the `attention` tensor is
         propagated to the next time step via the state and is used there.
         This flag only controls whether the attention mechanism is propagated
         up to the next cell in an RNN stack or to the top RNN output.
       initial_cell_state: The initial state value to use for the cell when
         the user calls `zero_state()`.  Note that if this value is provided
         now, and the user uses a `batch_size` argument of `zero_state` which
         does not match the batch size of `initial_cell_state`, proper
         behavior is not guaranteed.
       name: Name to use when creating ops.
       attention_layer: A list of `tf.layers.Layer` instances or a
         single `tf.layers.Layer` instance taking the context and cell output as
         inputs to generate attention at each time step. If None (default), use
         the context as attention at each time step. If attention_mechanism is a
         list, attention_layer must be a list of the same length. If
         attention_layers_size is set, this must be None.
     Raises:
       TypeError: `attention_layer_size` is not None and (`attention_mechanism`
         is a list but `attention_layer_size` is not; or vice versa).
       ValueError: if `attention_layer_size` is not None, `attention_mechanism`
         is a list, and its length does not match that of `attention_layer_size`;
         if `attention_layer_size` and `attention_layer` are set simultaneously.
     """
	 ......

   def call(self, inputs, state):
     """Perform a step of attention-wrapped RNN.

     - Step 1: Mix the `inputs` and previous step's `attention` output via
       `cell_input_fn`.
     - Step 2: Call the wrapped `cell` with this input and its previous state.
     - Step 3: Score the cell's output with `attention_mechanism`.
     - Step 4: Calculate the alignments by passing the score through the
       `normalizer`.
     - Step 5: Calculate the context vector as the inner product between the
       alignments and the attention_mechanism's values (memory).
     - Step 6: Calculate the attention output by concatenating the cell output
       and context through the attention layer (a linear layer with
       `attention_layer_size` outputs).

     Args:
       inputs: (Possibly nested tuple of) Tensor, the input at this time step.
       state: An instance of `AttentionWrapperState` containing
         tensors from the previous time step.

     Returns:
       A tuple `(attention_or_cell_output, next_state)`, where:

       - `attention_or_cell_output` depending on `output_attention`.
       - `next_state` is an instance of `AttentionWrapperState`
          containing the state calculated at this time step.

     Raises:
       TypeError: If `state` is not an instance of `AttentionWrapperState`.
     """
     if not isinstance(state, AttentionWrapperState):
       raise TypeError("Expected state to be instance of AttentionWrapperState. "
                       "Received type %s instead."  % type(state))
     # Step 1: Calculate the true inputs to the cell based on the
     # previous attention value.
     cell_inputs = self._cell_input_fn(inputs, state.attention)
     cell_state = state.cell_state
     cell_output, next_cell_state = self._cell(cell_inputs, cell_state)

     cell_batch_size = (
         cell_output.shape[0].value or array_ops.shape(cell_output)[0])
     error_message = (
         "When applying AttentionWrapper %s: " % self.name +
         "Non-matching batch sizes between the memory "
         "(encoder output) and the query (decoder output).  Are you using "
         "the BeamSearchDecoder?  You may need to tile your memory input via "
         "the tf.contrib.seq2seq.tile_batch function with argument "
         "multiple=beam_width.")
     with ops.control_dependencies(
         self._batch_size_checks(cell_batch_size, error_message)):
       cell_output = array_ops.identity(
           cell_output, name="checked_cell_output")

     if self._is_multi:
       previous_attention_state = state.attention_state
       previous_alignment_history = state.alignment_history
     else:
       previous_attention_state = [state.attention_state]
       previous_alignment_history = [state.alignment_history]
     all_alignments = []
     all_attentions = []
     all_attention_states = []
     maybe_all_histories = []
     for i, attention_mechanism in enumerate(self._attention_mechanisms):
       attention, alignments, next_attention_state = _compute_attention(
           attention_mechanism, cell_output, previous_attention_state[i],
           self._attention_layers[i] if self._attention_layers else None)
       alignment_history = previous_alignment_history[i].write(
           state.time, alignments) if self._alignment_history else ()

       all_attention_states.append(next_attention_state)
       all_alignments.append(alignments)
       all_attentions.append(attention)
       maybe_all_histories.append(alignment_history)

     attention = array_ops.concat(all_attentions, 1)
     next_state = AttentionWrapperState(
         time=state.time + 1,
         cell_state=next_cell_state,
         attention=attention,
         attention_state=self._item_or_tuple(all_attention_states),
         alignments=self._item_or_tuple(all_alignments),
         alignment_history=self._item_or_tuple(maybe_all_histories))

     if self._output_attention:
       return attention, next_state
     else:
       return cell_output, next_state

```
