# Self-host Gorilla (API Appstore for LLMs) on any cloud with one click

This post shows how to **use** [**SkyPilot**](https://github.com/skypilot-org/skypilot) **to host the [Gorilla](https://gorilla.cs.berkeley.edu/) chatbot** with just **one CLI command**.

It will automatically perform the following:

- **Get a beefy GPU instance** on AWS, GCP, Azure, or Lambda Labs
- **Set up the instance** (download weights, install requirements in a Conda env, etc.)
- **Launch a chatbot interface** that we can connect to through our laptop's browser

...and it does so while abstracting away all of the above infra burden and minimizing costs.

## Background

[**Gorilla**](https://gorilla.cs.berkeley.edu/) is a LLM that can provide appropriate API calls. It is trained on three massive machine learning hub datasets: Torch Hub, TensorFlow Hub and HuggingFace. We are rapidly adding new domains, including Kubernetes, GCP, AWS, OpenAPI, and more. Zero-shot Gorilla outperforms GPT-4, Chat-GPT and Claude. Gorilla is extremely reliable, and significantly reduces hallucination errors.
<img src="https://gorilla.cs.berkeley.edu/assets/img/gorilla_method.png" width=720 alt="Run Gorilla LLM chatbots on any cloud with SkyPilot"/>

[**SkyPilot**](https://github.com/skypilot-org/skypilot) is an open-source framework from UC Berkeley for seamlessly running machine learning on any cloud. With a simple CLI, users can easily launch many clusters and jobs, while substantially lowering their cloud bills. Currently, [Lambda Labs](https://skypilot.readthedocs.io/en/latest/getting-started/installation.html#lambda-cloud) (low-cost GPU cloud), [AWS](https://skypilot.readthedocs.io/en/latest/getting-started/installation.html#aws), [GCP](https://skypilot.readthedocs.io/en/latest/getting-started/installation.html#gcp), and [Azure](https://skypilot.readthedocs.io/en/latest/getting-started/installation.html#azure) are supported. See [docs](https://skypilot.readthedocs.io/en/latest/) to learn more.
<img src="https://raw.githubusercontent.com/skypilot-org/skypilot/master/docs/source/images/cloud-logos-light.png" width=720 alt="Run Gorilla LLM chatbots on any cloud with SkyPilot"/>

## Steps

All YAML files used below live in [the SkyPilot repo](https://github.com/skypilot-org/skypilot/tree/master/llm/gorilla).

0. Install SkyPilot and [check that cloud credentials exist](https://skypilot.readthedocs.io/en/latest/getting-started/installation.html#cloud-account-setup):

   ```bash
   pip install "skypilot[aws,gcp,azure,lambda]"  # pick your clouds
   sky check
   ```

   <img src="https://i.imgur.com/7BUci5n.png" width="485" alt="`sky check` output showing enabled clouds for SkyPilot"/>

1. Get the [example folder](https://github.com/skypilot-org/skypilot/tree/master/llm/gorilla):

   ```bash
   git clone https://github.com/skypilot-org/skypilot.git
   cd skypilot/llm/gorilla
   ```

2. a. **Run the following command**:

   ```bash
   sky launch gorilla.yaml -c gorilla -s
   ```

   This will download the 7B Gorilla model to the cloud instance. The setup process can take up to 10 minutes.

   b. You will see a confirmation prompt like the following:
   <img src="https://i.ibb.co/rQLfkFh/Screenshot-2023-11-12-201507.png" alt="`sky launch` output showing 8x A100 GPUs on different clouds"/>
   <p align="center"><sub>SkyPilot automatically finds the cheapest A100 GPU across clouds and regions.</sub></p>
   You may see fewer clouds because SkyPilot uses only the clouds you have access to.

   See [below](#more-commands-to-try) for many more commands to run!

3. Open another terminal and run:

   ```bash
   ssh -L 7681:localhost:7681 gorilla
   ```

4. Open http://localhost:7681 in your browser and start chatting!
   <img src="https://i.ibb.co/7SxWky7/Screenshot-2023-11-12-203002.png" alt="Gorilla chatbot running on the cloud via SkyPilot"/>

## More commands to try

**Launch on different clouds** with `--cloud` (optional):
| **Cloud** | **Command** |
| ----------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Launch on the cheapest cloud/region (automatically chosen!) | `sky launch gorilla.yaml -c gorilla -s ` |
| Launch on Lambda Labs | `sky launch gorilla.yaml --cloud lambda -c gorilla -s ` |
| Launch on GCP | `sky launch gorilla.yaml --cloud gcp -c gorilla -s ` |
| Launch on AWS | `sky launch gorilla.yaml --cloud aws -c gorilla -s ` |
| Launch on Azure | `sky launch gorilla.yaml --cloud azure -c gorilla -s` |

**Use spot instances** to save >3x costs with `--use-spot`:

```bash
sky launch gorilla.yaml --use-spot <other args>
```

To see details about these flags, see [CLI docs](https://skypilot.readthedocs.io/en/latest/reference/cli.html) or run `sky launch -h`.

## Cleaning up

When you are done, you can stop or tear down the cluster:

- **To stop the cluster**, run
  ```bash
  sky stop gorilla  # or pass your custom name if you used "-c <other name>"
  ```
  You can restart a stopped cluster and relaunch the chatbot (the `run` section in YAML) with
  ```bash
  sky launch gorilla.yaml -c gorilla --no-setup
  ```
  Note the `--no-setup` flag: a stopped cluster preserves its disk contents so we can skip redoing the setup.
- **To tear down the cluster** (non-restartable), run
  `bash
sky down gorilla  # or pass your custom name if you used "-c <other name>"
`
  **To see your clusters**, run `sky status`, which is a single pane of glass for all your clusters across regions/clouds.

To learn more about various SkyPilot commands, see [Quickstart](https://skypilot.readthedocs.io/en/latest/getting-started/quickstart.html).

## Why SkyPilot?
As open LLMs become more powerful, bigger, and more compute-hungry, the demand of **flexibly finetuning and running them on a variety of cloud compute** will dramatically increase.
And that is where SkyPilot comes in. This example shows three major benefits of using SkyPilot to run ML projects on the cloud:

**Cloud portability & productivity**: We've wrapped an existing ML project and launched it to the cloud of your choosing using a simple YAML and one command. Interacting with a simple CLI, users get _cloud portability_ with one argument change.

SkyPilot also improves ML users' productivity of using the cloud. There's no need to learn different clouds' consoles or APIs. No need to figure out the right instance types. And there's **no change to the actual project code** for it to run.

**Higher GPU availability**: If a region or a whole cloud runs out of GPUs (increasingly common in today's Large Models race), the only solution other than waiting is to **go to more regions and clouds**.

![SkyPilot's auto-failover across regions and clouds to improve GPU availability](https://i.imgur.com/zDl2zob.png)

<p align="center"><sub>SkyPilot's auto-failover. Regions from all enabled clouds are sorted by price and attempted in that order. If a launch request is constrained to use a specific cloud, only that cloud's regions are used in failover. Ordering in figure is illustrative and may not reflect the most up-to-date prices.</sub></p>

SkyPilot's `sky launch` command makes this entirely automatic. It performs _auto-failover_ behind the scenes. For each request the system loops through all enabled regions (or even clouds) to find available GPUs, and does so in the cheapest-price order.

**Reducing cloud bills**: GPUs can be very expensive on the cloud. SkyPilot reduces ML teams' costs by supporting

- low-cost GPU cloud (Lambda; >3x cheaper than AWS/Azure/GCP)
- spot instances (>3x cheaper than on-demand)
- automatically choosing the cheapest cloud/region/zone
- auto-stopping & auto-termination of instances ([docs](https://skypilot.readthedocs.io/en/latest/reference/auto-stop.html))

## Recap

Congratulations! You have used SkyPilot to launch a gorilla chatbot on the cloud with just one command. The system automatically handles setting up instances and it offers cloud portability, higher GPU availability, and cost reduction.

Gorilla chatbots are just one example app. To leverage these benefits for your own ML projects on the cloud, we recommend the [Quickstart guide](https://skypilot.readthedocs.io/en/latest/getting-started/quickstart.html).

_Feedback or questions? Want to run other LLM models?_ Feel free to drop a note to the SkyPilot team on [GitHub](https://github.com/skypilot-org/skypilot/) or [Slack](http://slack.skypilot.co/) and we're happy to chat!
