# 2023-ecosystem-paper
## Introduction
Neurophysiology research relies heavily on the acquisition and analysis of complex, multimodal, and increasingly large datasets, necessitating robust standards for data organization, sharing, and archival. The Neurodata Without Borders (NWB) format effectively archives neurophysiology datasets by storing contextual metadata information alongside all primary data objects, ensuring comprehensive and reliable long-term data interpretability. NWB also binds together all of the different relevant modalities for a given experiment, such as simultaneous behavior and electrophysiology, which are typically scattered across multiple different file formats. This enables robust reanalysis, since each NWB file has all of the relevant information about a given experimental session. The Distributed Archives for Neurophysiology Data Integration (DANDI) platform encourages data sharing and collaboration by hosting NWB-formatted datasets up to TBs in size at no cost to users. DANDI provides researchers a free way to meet modern NIH data sharing requirements, effectively archive their data, and take advantage of continually developing visualization tools. The NWB format and DANDI archive not only facilitate the exchange of datasets among researchers but also streamline data for software tool developers. The highly standardized nature of the NWB format together with the easy accessibility of the datasets in the DANDI archive make the platform an ideal sandbox for emerging software domains such as electrophysiological spike sorting, calcium imaging segmentation, and behavioral pose estimation.

Despite the numerous advantages offered by NWB and DANDI, widespread adoption is hindered by a multitude of challenges, each presenting a distinct hurdle for laboratories seeking to seamlessly integrate these resources into their research workflows. First, adopting NWB requires at least a summary understanding of its data organization and principles, the sheer scale of which can be daunting for those unfamiliar with its structure. This learning curve poses a substantial obstacle, slowing down the assimilation of NWB standards into the daily practices of research labs. In addition, the utilization of data Application Programming Interfaces (APIs), such as _pyNWB_ or _MatNWB_, demands a level of programming expertise that may be beyond the reach of researchers with limited coding experience. This technical barrier creates an accessibility gap, restricting the adoption of NWB to those proficient in programming or placing an outsized burden on those, often junior, lab members with the necessary expertise.

If labs do make the decision to adopt the NWB standard, they often find themselves motivated to convert their backlogged data to the NWB format. Recognizing the advantages of NWB, labs aim to ensure that their extensive historical datasets align with this standardized framework, enhancing data interoperability, collaboration, and long-term accessibility. However, this effort tends to be a substantial bottleneck in the transition to NWB. The accumulation of historical data, often stored in diverse and proprietary formats, poses a formidable challenge, as researchers must now navigate through and transform these datasets to comply with NWB. The sheer volume of legacy data, combined with its inherent heterogeneity, turns the conversion process into a labor-intensive endeavor. Further hampering adoption efforts is the dearth of individual incentives for researchers to prioritize this time-consuming task. In the fast-paced environment of scientific research, where immediate outcomes and results often take precedence, dedicating time and resources to the retrospective conversion of historical data may not be perceived as an urgent priority for individual researchers. The absence of tangible, career-advancing rewards or incentives further hampers the motivation to address this critical aspect of NWB adoption.

One common approach to this problem is to hire or leverage dedicated research software engineers (RSEs) [cite Ritt Group, NYU, Princeton] or contract with data management/consulting companies (ex. DataJoint) to convert the data. But, since these resources tend to be distributed across many different universities and institutions, much of the work may be duplicated. Common conversion frameworks and tools can help accelerate development and make conversions easier for all stakeholders.

Just as labs frequently seek to convert backlogged data, they are also often motivated to establish conversion pipelines in advance of experimental data collection, so that newly generated data can be automatically converted into NWB with minimal duplicated work. For example, a lab may develop a conversion pipeline on existing data, and then try to generalize the pipeline so that it can be used for any follow up experiments. Automatically converting diverse neurophysiology data formats presents challenges due to the wide variety of modalities involved. Even a single lab may employ voltage recording, optical imaging, optogenetics, behavior, and more, each with its own software-dependent formats lacking standardization. A lab-specific conversion pipeline must take all the possible options into account to properly future-proof, or adapt itself to every new experiment that adds or removes a piece of software or hardware. These considerations make conversion pipelines using _PyNWB_ or _MatNWB_ alone difficult to develop and even harder to maintain, and are often cited as a specific pain point [cite A perspective on neuroscience data standardization with NWB].

To mitigate these challenges and promote wider adoption of NWB, we have developed a sophisticated software suite comprising three core tools. The first, _NWB Inspector_, is a Python-based tool that ensures the validity of NWB files, enhancing the reliability of their metadata when uploading to sharing platforms like DANDI. The second, _NeuroConv_, is a software package that facilitates the conversion of neurophysiology data from various proprietary formats to the NWB standard. Its primary purpose is to make it easy to develop automatic conversion pipelines that ingest data from all recording and processing sources used in a neurophysiology experiment. The third tool, _NWB GUIDE (Graphical User Interface for Data Entry)_, is an interactive desktop application that walks users through the process of converting their data to the NWB standard without requiring any programming whatsoever.

In summary, our software suite addresses the multifaceted challenges associated with NWB adoption, offering tools that enhance data integrity, automate conversion processes, and provide accessible solutions for neurophysiologists at all programming skill levels. Through these innovations, we aim to catalyze the widespread implementation of NWB standards, fostering collaboration and advancing neurophysiological research.

## Results
### NWBInspector
It can be time-consuming and error-prone to manually inspect all of the NWB files in a dataset, especially as datasets grow very large. DANDI datasets frequently occupy hundreds of GBs to TBs of disk space, spread across thousands of individual NWB files [cite DANDI]. This challenge is compounded for new users who may lack familiarity with NWB best practices, further hampering the effectiveness of manual inspection. To address this problem, we have developed _NWBInspector_, a Python-based package that assesses the quality of NWB files based on their compliance with established best practices.

After installation, _NWBInspector_ can be used very easily from the command line. Simply run
```bash
    nwbinspector path/to/my/data.nwb
```
to inspect a single NWB file or
```bash
        nwbinspector path/to/my/data/dir/
```
to inspect all the NWB files in a given directory.

This command will produce a report of all the issues flagged by _NWBInspector_ directly in the terminal, ordered by importance as shown below.
[Example output]

Unlike the _PyNWB_ validator, which checks for strict schema compliance, _NWBInspector_ only serves as an automatic data review aid. This means that, in an effort to catch as many potential mistakes as possible, _NWBInspector_ may raise warnings for intentional deviations from standard best practices. Conversely, if a user makes a mistake that does not violate any established best practices and it passes the _PyNWB_ validator, _NWBInspector_ might not catch that mistake. This structure leads to a trade-off between comprehensive inspection, which catches as many potential mistakes as possible, and an unobtrusive inspection process that only highlights truly problematic choices rather than intentional but unorthodox decisions.

Our solution to this trade-off involves splitting the results into three tiers of warnings/errors. The first tier is for best practice violations, which are clear violations of known best practices, such as negative spike times. The second tier is for critical issues, which are potentially critical mistakes, such as zero-rate timestamps. The third tier is for best practice suggestions, which are recommendations for improving the NWB file, such as adding an informative description. This approach allows our inspection process to be as comprehensive as possible while minimizing disruptions from ignoring specific best practice suggestions.

Power users of NWBInspector often seek more direct control of the inspection process and its outputs. To enable more flexibility when using _NWBInspector_, we provide a simple Python API with all the same functionality as the CLI, along with some more advanced features. Just like the CLI, the Python API can be employed to inspect individual NWB files and entire directories of NWB files with the `nwbinspector.inspect_nwbfile()` function and `nwbinspector.inspect_all()` function respectively. Additionally, the Python API allows users to inspect NWB files directly in memory with the `nwbinspector.inspect_nwbfile_object()` function. Since both the `inspect_nwbfile` and the `inspect_all` functions return generators, Python users can employ these functions in more sophisticated ways, such as implementing early stopping if a particular message is raised. These features empower advanced users to fully leverage _NWBInspector_.
