### no_std

The application of #![no_std] at the crate level in Rust denotes a deliberate choice to link with the core crate rather than the std crate. This choice is significant in scenarios where the program aims for minimal dependencies and operates in environments without a full-fledged standard library. In this context, the [libcore](https://doc.rust-lang.org/core/) crate serves as a versatile foundation, offering a platform-agnostic subset of the standard library.

Libcore focuses on essential language constructs like floats, strings, and slices, providing crucial APIs for these primitives. Additionally, it exposes interfaces for low-level processor features, including atomic operations and SIMD instructions. However, it deliberately omits APIs related to platform-specific integrations, ensuring its adaptability to diverse environments.

It's worth noting that due to these characteristics, code marked as no_std and utilizing [libcore](https://doc.rust-lang.org/core/) finds applications beyond standard Rust environments. Yet, it is essential to recognize that such code is not suitable for tasks requiring direct interaction with specific platform features, making it unsuitable for bootstrapping activities such as building bootloaders, firmware, or kernels. This intentional abstraction provides a flexible foundation for projects that prioritize efficiency, minimalism, and portability.

### Overview

| feature                                                   | no\_std | std |
|-----------------------------------------------------------|--------|-----|
| heap (dynamic memory)                                     |   *    |  ✓  |
| collections (Vec, BTreeMap, etc)                          |  **    |  ✓  |
| stack overflow protection                                 |   ✘    |  ✓  |
| runs init code before main                                |   ✘    |  ✓  |
| libstd available                                          |   ✘    |  ✓  |
| libcore available                                         |   ✓    |  ✓  |
| writing firmware, kernel, or bootloader code              |   ✓    |  ✘  |

\* Only if you use the `alloc` crate and use a suitable allocator like [alloc-cortex-m].

\** Only if you use the `collections` crate and configure a global default allocator.

\** HashMap and HashSet are not available due to a lack of a secure random number generator.



## Bare Metal Environments

In the realm of bare metal programming, your program begins its execution without any pre-loaded code. Unlike environments supported by an operating system, a bare metal setup lacks the infrastructure to load the standard library. Here, the program, along with the crates it utilizes, solely interacts with the hardware—running in a "bare metal" state.

To instruct Rust not to load the standard library, the no_std attribute is employed. In this context, the platform-agnostic elements of the standard library are accessible through libcore. Notably, [libcore](https://doc.rust-lang.org/core/) deliberately excludes components that may not be universally suitable in an embedded environment. A prime example is the omission of a memory allocator for dynamic memory allocation.

In scenarios where additional functionalities, such as a memory allocator, are needed, numerous crates are available to fulfill these requirements. These crates extend the capabilities of bare metal programming by providing specific features while maintaining compatibility with the constraints of embedded systems.

Rust, with its emphasis on memory safety and low-level control, emerges as a valuable tool for bare metal programming. Its no_std feature, coupled with libcore, enables developers to harness the power of Rust in environments where direct hardware interaction is paramount, allowing for efficient, secure, and tailored solutions in the realm of embedded systems.
