# DiT-WM: Scalable Diffusion Transformer World Model for Model-based RL

### Abstract:
There have been significant improvements of world-models recently. take Genie 3 or the world-model of [DreamerV3](https://arxiv.org/pdf/2301.04104), both by researchers at Google DeepMind, they are exceptional in certain aspects. Genie 3 is incredibly good at consistency, while DreamerV3 is efficient, while still being robust to train a highly capable policy. Yet they also have weaknesses, Genie 3 is not readily applicable for the context of training an agent in a specific environment, and DreamerV3 RSSM consistency degrade over long horizons as error accumulate. Other works have tried Diffusion for world models, for example the work at [Princeton and Meta](https://arxiv.org/pdf/2402.03570), but have not yet tried in an online learning environment, while being computationally demanding. In this repository I propose the idea of using Diffusion Transformer as a World Model for model-based Reinforcement Learning, along with a training pipeline to make the idea feasible for online policy optimization.

### Inspiration:
The idea came from the fact that image and video generation models, they generate a single or sequence of images based on a condition of text and the diffused latent/image. And my idea is that replacing the text condition with previous state/action condition, and having the model denoise from a noise latent to generate the next state. And to make them actually fast, we can use latent diffusion, where the model denoise the latent that will eeventually be decompressed into an image using the decoder form a VAE.

### Specific Details:
We might embed the individual states and action at each time step, resulting in (B, T, S_z) and (B, T, A_z), concatenating the vectors together, resulting in a sequence of shape (B, T, S_z + A_z), treating them as a sequence of "tokens", similarly to how text gets tokenized and embedded, but in this case vectors are representations of states and actions. Add positional embedding as usual, and feed into the Diffusion Transformer as condition.

And to make it computationally feasible for real world usage, such as training an actual agent in an environment, I propose a training pipeline with 3 main steps. First training the agent with the fast, efficient world model such as RSSM, the world model will eventually reach its maximum capacity after a while, but the main purpose of this step is to gather large amount of data of the environment's dynamics, while also conveniently optimizing the policy. Next, using the vast data collected from the first step, we can freeze the policy/value, encoder and decoder networks. We would encode all the raw observations into latent (z_t), and train the DiT-WM without updating any other networks.

with latent at time step t
the input would be z_t
target is z_(t+1), while the prediction would be z̄_(t+1)
we can also extend it to z̄_(t+n) and combine the losses

After done pre-training the DiT-WM, we would distill it furhter into a one-step or few-step model to make it usable for RL.

Afterwards, the 3rd step is to replace the old, fast world model (RSSM) with the new, robust DiT world model.

Other approaches such as mixed-precision training could also be used to improve efficiency.

### Expected Results:
While this is an interesting idea, I do not guarantee this concept to be successfully at first, there would be unexpected elements that I did not take into account and could affect the final results. But if successful, I believe this architecture would be a leap forward, improving upon the current works. I expect a successful DiT-WM would be able to "imagine" long, high-fidelity trajectories, plus with the stochastic nature of Diffusion models, the DiT-WM could imagine multiple trajectories for each action, take into account the stocasticity nature of the environment, without averaging, blending all the possibilities together, which results in a blurry imagination that could be physically impossible for the environment.

### Side notes:
This readme file was not written by a high-school student alone, I might have not explained this in full technical details and there could be some fundamental errors in the idea. My way of writing wouldn't be suitable for a full technical research paper or formal a formal report.

### References
-   Ball, P. J., Bauer, J., Belletti, F., et al. (2025). *Genie 3: A New Frontier for World Models*.
    [Link to source]([https://arxiv.org/abs/XXXX.XXXXX](https://deepmind.google/discover/blog/genie-3-a-new-frontier-for-world-models/))
-   Ding, Z., Zhang, A., Tian, Y., & Zheng, Q. (2024). *Diffusion World Model: Future Modeling Beyond Step-by-Step Rollout for Offline Reinforcement Learning*. ArXiv.
    [https://arxiv.org/abs/2402.03570](https://arxiv.org/abs/2402.03570)
-   Hafner, D., Pasukonis, J., Ba, J., & Lillicrap, T. (2023). *Mastering Diverse Domains through World Models*. ArXiv.
    [https://arxiv.org/abs/2301.04104](https://arxiv.org/abs/2301.04104)
-   Peebles, W., & Xie, S. (2022). *Scalable Diffusion Models with Transformers*. ArXiv.
    [https://arxiv.org/abs/2212.09748](https://arxiv.org/abs/2212.09748)
-   Rombach, R., Blattmann, A., Lorenz, D., Esser, P., & Ommer, B. (2021). *High-Resolution Image Synthesis with Latent Diffusion Models*. ArXiv.
    [https://arxiv.org/abs/2112.10752](https://arxiv.org/abs/2112.10752)
