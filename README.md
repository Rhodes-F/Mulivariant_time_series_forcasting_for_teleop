# Multivariant time series forcasting for teleoperation
## Required Hardware
- At least 2 HTC Vive Base Stations
- A VR headset compatible with open VR 
- HaptX gloves 
- A computer meeting the minimum specifications mentioned [here](http://support.haptx.com/docs/sdk/page_system_requirements.html) 
## Unity Project
This project includes a simulation to run the HaptX glove and stack blocks with haptic feedback in VR. This project was built on top of the HaptX Starter project, with help from the example made by Argonne National Laboratory. To run this project, download the unity project from this repository. Next install unity 2021.3.19f1, steam VR, and any software needed to get your VR headset working. We used a Varjo Aero Headset however any headset that is compatible with Open VR should work. For best results, the HaptX hand configuration should also be used to make a model of the user's hand. Once all the required software is installed, to run the project begin by setting up the HTC Vive Base Stations and pairing the tracker on the glove using steam VR. Next open the project using unity hub and run it by pressing the run button at the top of the scene. Once the project is running, the user should be able to see the scene which includes 3 blocks, a table and the model of the hand that you have made (if no model is made the default will be used). The project is designed to track the three blocks (position:x,y,z,angle:quaternion), and the position on the palm of the hand and each of the finger tips (also (position:x,y,z,angle:quaternion)). To use the hand tracking and saving, you must link each of the fingers individually. To do this open the right-hand model and drag index4, middle4, ring4, pinky4, and root to the respective fingers in the index_tip, middle_tip, ring_tip, pinky_tip, and palm game objects. You also must drag the root object into the palm slot. Once this is done, the unity project is ready to be used. The blocks can be reset by pressing "R" on the keyboard. Pressing "R" will also end the trail and save the next motions in a new CSV. Once this is done and you have gathered the desired amount of data. Simply stop the simulation by pressing the play button once again and then move the data that was gathered from the data folder to a different folder. This is important because the unity script will overwrite any data that is left in the data folder if this is not done.
## Data Set
The data is collected into a data folder in unity. To collect the data, the user simply stacks the three blocks in red green blue order (bottom to top) and then presses the r key to get a new trial. Each trial randomizes the blocks locations. Data for each trial is collected into a CSV at 50 HZ and is formatted according to 
    Datetime,cube_0_position,cube_0_rotation,cube_1_position,cube_1_rotation,cube_2_position,cube_2_rotation,index_tip_pos,index_tip_rot,middle_tip_pos,middle_tip_rot,ring_tip_pos,ring_tip_rot,pinky_tip_pos,pinky_tip_rot,thumb_tip_pos,thumb_tip_rot,palm_pos,palm_rot
    
For our dataset, we stacked blocks for 974 trials, each roughly 3 seconds in length. Data was collected starting from both a sitting and standing position. If a block tower fell over, it was simply restacked. If two blocks randomly spawned on top of each other, they would be left alone if already in the proper order. Otherwise they would be unstacked and then restacked. In the event a block fell off the table, we would wait for a significant amount of time to make the trial clearly identifiable by its larger file size. The trial will be removed.
## Data Format and cleaning
The data that was gathered was also in a format that was not easily read by the transformer. To mend this, we wrote many scripts to clean, and error check the data. These scrips are included and labeled as data cleaning. The purpose of these scripts is to first check and remove any parenthesis that were added during collection. They also check to make sure that each trial has the correct number of rows and columns and if they do not those trials are removed. Finally, all of the data is combined into one file and the positions of the beginnings of each file are recorded as a text file. This allows the transformer to read the data and by recording the beginnings of each trial we can ensure that the transformer’s window does not overlap across multiple trials.

## Transfomer
[Fedformer Paper](https://arxiv.org/abs/2201.12740)
[Fedformer Github](https://github.com/MAZiqing/FEDformer)

The model is trained with a Frequency Enhanced Decomposed Transformer which works on multivariate time series data.

To train the model, run main2.py after installing the requirements.txt. Training arguments can be passed in through the command line.
The loss curves and model checkpoints will output to a checkpoints folder in scipts.
Data is read from ./data/formatted_data.csv and row_counts.txt is used to ensure the transformer does not learn a sequence across more than one episode of training.

    parser.add_argument('--is_training', type=int, default=1)
    parser.add_argument('--root_path', type=str, default='../data')
    parser.add_argument('--data_path', type=str, default='formatted_data.csv')
    parser.add_argument('--task_id', type=str, default='Exchange')
    parser.add_argument('--model', type=str, default='FEDformer')
    parser.add_argument('--data', type=str, default='custom')
    parser.add_argument('--features', type=str, default='M')
    parser.add_argument('--seq_len', type=int, default=128)
    parser.add_argument('--label_len', type=int, default=64)
    parser.add_argument('--pred_len', type=int, default=150)
    parser.add_argument('--e_layers', type=int, default=2)
    parser.add_argument('--d_layers', type=int, default=1)
    parser.add_argument('--factor', type=int, default=3)
    parser.add_argument('--enc_in', type=int, default=63)
    parser.add_argument('--dec_in', type=int, default=63)
    parser.add_argument('--c_out', type=int, default=63)
    parser.add_argument('--des', type=str, default='Exp')
    parser.add_argument('--itr', type=int, default=1)

    # supplementary config for FEDformer model
    parser.add_argument('--version', type=str, default='Wavelets',
                        help='for FEDformer, there are two versions to choose, options: [Fourier, Wavelets]')
    parser.add_argument('--mode_select', type=str, default='random',
                        help='for FEDformer, there are two mode selection method, options: [random, low]')
    parser.add_argument('--modes', type=int, default=64, help='modes to be selected random 64')
    parser.add_argument('--L', type=int, default=3, help='ignore level')
    parser.add_argument('--base', type=str, default='legendre', help='mwt base')
    parser.add_argument('--cross_activation', type=str, default='tanh',
                        help='mwt cross atention activation function tanh or softmax')

    parser.add_argument('--target', type=str, default='OT', help='target feature in S or MS task')
    # try to use .001s if works or dont use timef for embed
    parser.add_argument('--freq', type=str, default='b',
                        help='freq for time features encoding, options:[s:secondly, t:minutely, h:hourly, d:daily, '
                             'b:business days, w:weekly, m:monthly], you can also use more detailed freq like 15min or 3h')
    parser.add_argument('--checkpoints', type=str, default='./checkpoints/', help='location of model checkpoints')

    # parser.add_argument('--cross_activation', type=str, default='tanh'

    # model define
    parser.add_argument('--d_model', type=int, default=256, help='dimension of model')
    parser.add_argument('--n_heads', type=int, default=8, help='num of heads')
    parser.add_argument('--d_ff', type=int, default=1024, help='dimension of fcn')
    parser.add_argument('--moving_avg', default=[11,25], help='window size of moving average')
    parser.add_argument('--distil', action='store_false',
                        help='whether to use distilling in encoder, using this argument means not using distilling',
                        default=True)
    parser.add_argument('--dropout', type=float, default=0.05, help='dropout')
    parser.add_argument('--embed', type=str, default='timeF',
                        help='time features encoding, options:[timeF, fixed, learned]')
    parser.add_argument('--activation', type=str, default='gelu', help='activation')
    parser.add_argument('--output_attention', action='store_true', help='whether to output attention in ecoder')
    parser.add_argument('--do_predict', action='store_true', help='whether to predict unseen future data',default=False)

    # optimization
    parser.add_argument('--num_workers', type=int, default=10, help='data loader num workers')
    parser.add_argument('--train_epochs', type=int, default=6, help='train epochs')
    parser.add_argument('--batch_size', type=int, default=16, help='batch size of train input data')
    parser.add_argument('--patience', type=int, default=3, help='early stopping patience')
    parser.add_argument('--learning_rate', type=float, default=0.0001, help='optimizer learning rate')
    parser.add_argument('--loss', type=str, default='mse', help='loss function')
    parser.add_argument('--lradj', type=str, default='type2', help='adjust learning rate')
    parser.add_argument('--use_amp', action='store_true', help='use automatic mixed precision training', default=False)

    # GPU
    parser.add_argument('--use_gpu', type=bool, default=True, help='use gpu')
    parser.add_argument('--gpu', type=int, default=0, help='gpu')
    parser.add_argument('--use_multi_gpu', action='store_true', help='use multiple gpus', default=False)
    parser.add_argument('--devices', type=str, default='0,1', help='device ids of multi gpus')
