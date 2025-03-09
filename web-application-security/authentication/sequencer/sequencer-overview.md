# Sequencer Overview

#### **Introduction to Burp Suite Sequencer**

Burp Suite’s **Sequencer** is a tool designed to analyze the randomness of tokens and session identifiers used in web applications. Many applications rely on unique and unpredictable session tokens for authentication and maintaining user sessions. If these tokens are not sufficiently random, an attacker can exploit weak token generation mechanisms to predict valid session identifiers and potentially hijack user accounts.

**How is Sequencer Used?**\
Sequencer is used to perform **statistical analysis** on session tokens, CSRF tokens, or other random values used for authentication and session management. It captures a large number of tokens, evaluates their randomness, and generates an entropy analysis report. This report highlights patterns or weaknesses in the token generation process, helping security professionals determine if an application’s session management is vulnerable.

**Key Features of Sequencer:**

* **Live Token Capture:** Automatically collects and analyzes tokens from an application.
* **Statistical Randomness Tests:** Assesses entropy levels and identifies patterns in token structure.
* **Character-Level and Bit-Level Analysis:** Evaluates the predictability of individual characters within tokens.
* **Custom Decoding Options:** Supports decoding mechanisms like Base64 and hexadecimal for better analysis.

**Common Use Cases:**

* **Session Token Analysis:** Checking if session IDs are predictable or follow a weak pattern.
* **CSRF Token Evaluation:** Determining if anti-CSRF tokens are sufficiently random.
* **Authentication Mechanism Testing:** Verifying whether “stay logged in” or API authentication tokens are securely implemented.

A weak session management system can lead to **session hijacking, brute-force attacks, and unauthorized account access**. By leveraging **Sequencer**, security professionals can assess and mitigate these risks, ensuring applications use strong, unpredictable session tokens.
