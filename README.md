# Flatland

As the file is too large, it is unable to be uploaded to git hub. 

Instead go to https://drive.google.com/drive/folders/1MBfOS-7na8c3yDXrXznyc0HTpvkw3Dt1?usp=sharing to download the file.

Once downlaoded, run the following to setup the project:

    cd "<project directory>"
    py -3.7 -m venv flatenv
    flatenv\Scripts\activate
    pip install wheel
    pip install -r requirements_dev.txt
    cd src
    python flatland_demo.py

Command-line instruction to install PyTorch using pip follow.

Windows / Linux:

	pip install torch==1.6.0+cpu -f https://download.pytorch.org/whl/torch_stable.html
  
Once done, run the eval files q1_priority_eval, q2_single_agent_eval and q3_priority_eval in the src folder for the program to run.

Logic are coded in q1_priority_plan, q2_single_agent_training and q3_priority_plan.
