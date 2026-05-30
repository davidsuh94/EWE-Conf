# EWE-Conf
Extreme Weather Event Confidence Calibration Benchmark

## Task Description
EWE-Conf is a benchmark created to evaluate the accuracy of LLMs and their ability to predict global weather patterns and extreme weather events and give a calculation of confidence. It is important for AI to accurately predict extreme weather, as they can highly impact hundreds to thousands of lives. A model should be reliable enough that scientists can trust the results to increase readiness and reduce disaster. A model that cannot give an estimated confidence in an output would not be meaningful, as scientists would not be able to calculate risk based on it.

Multiple LLMS (Claude, ChatGPT, Gemini) will be evaluated by comparing results to ground-truth. A total of 50 historical weather events were compiled and given the same prompt to predict the weather state (temperature, wind speed/direction, mean sea-level pressure) and give an extreme event prediction (yes/no + severity level) and a confidence level, given the weather state 6 hours prior.

These items were then scored against the ground-truth weather states and confidence levels, based on correctness, severity alignment, and calibration score.

## Data Schema
