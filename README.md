# cosmos-sdk-simapp-inflation

## 1. Set up local node

First of all we have to clone cosmos-sdk repo to local machine. We can do it by running the command:
```
git clone https://github.com/cosmos/cosmos-sdk.git
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/56624aef-ea09-48db-8fd3-bca72dde00f4)

After that we'll enter the directory with cloned repository and build the SimApp
```
cd cosmos-sdk/
make build
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/57774d1a-1759-47e9-90ab-ac17973d943c)
Now when build is completed let's initialize the SimApp
```
cd build/
./simd init test --chain-id 1
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/bea416fa-83d9-4ec8-b05e-c3d1f85a767e)

We will add a new key with the following command
```
./simd keys add marlen
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/b67a0291-70c3-400a-a2c2-2e563ad1ffac)

We will save address `cosmos1c6qqnssyraq6f85x77nk5ggjp83ux5z6hrzr5f` for future transactions

So now is time to create genesis account and transaction that will create a validator
```
./simd genesis add-genesis-account marlen 10000000000000000000000000stake
./simd genesis gentx marlen 1000000000stake --chain-id 1
```
![image_2024-05-24_17-13-44](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/3a68f366-585c-4e57-a604-001349132ce8)

Next we can generate our genesis file and save it as genesis.json file: 
```
./simd genesis collect-gentxs &> genesis.json
```
![image_2024-05-24_17-20-33](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/2724cf7c-86bd-45c6-8afa-0a07f39fb856)

Finally we can start our SimApp
```
./simd start
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/05bead3d-bb39-4ca9-a3f0-9e42786d4bda)


In output we can see blocks execution

![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/032f2b59-4ea8-45fb-b259-4a02d768be79)

## 2. Execute some transactions and show how inflation is changing

Firstly, let's open another terminal and run commands to get inflation at first few committed blocks 
```
./simd query mint inflation --height [block_number]
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/c11a7676-1441-4e6d-9854-6e753e315f2b)

If we'll calculate the differences between those inflation rates, we'll get that it's equal to 0.000000020597257079 in all cases

According to the code the inflation rate change per year is calculated like
```
inflationRateChangePerYear = (1 - bondedRatio/params.GoalBonded) * params.InflationRateChange
```

Let's check mint module's params:
```
./simd query mint params
```

![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/328c1477-0cc7-49ab-99c8-4452bc2a23cd)

Here we can see that `GoalBonded` is equal to 0.67 and inflation rate change is 0.13 (all decimals here multiplied by 10^18)

As we did only one genesis transaction and nothing more so `bondedRatio` should be almost equal to `0`, so approximately we can say that inflation rate change per year is equal to `0.13`. To calculate the inflation rate change per block we simply have to divide it by blocks per year which is listed on previous screen: `0.13 / 6311520 ≈ 0.000000020597257079`

Now when we know that bonded ratio affects inflation change let's make a transaction to delegate tokens to validator. Firstly we will get the address of validator by listing all validators:
```
./simd query staking validators
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/eeda1b34-7bbb-4017-ab7d-ca43c90ab9b2)


Here it is labeled as operator_address: ```cosmosvaloper1c6qqnssyraq6f85x77nk5ggjp83ux5z6jhkkc6```

So now we'll delegate 1000000000000000000stake to this validator:
```
./simd tx staking delegate cosmosvaloper1c6qqnssyraq6f85x77nk5ggjp83ux5z6jhkkc6 1000000000000000000stake --from cosmos1c6qqnssyraq6f85x77nk5ggjp83ux5z6hrzr5f
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/ac6c8a08-3c9b-4f56-a0bd-824af8c5c7b7)

So now let's measure inflation again as we did before

![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/0ecce5a2-37b6-4949-9dde-7cab9684110a)

As we can see now inflation rate change is `0.000000020597254005` which is less than it was before

Now because we bonded `1000000000000000000stake` which is `0.0000001` of genesis amount we can estimate bonded ratio as `0.0000001` (because not so much time has passed since genesis and supply hasn't become really bigger). So now to calculate new inflation rate change per block we'll take our previous change `0.000000020597257079` and multiply it by `(1 - 0.0000001 / 0.67) ≈ 0.9999998507462686` which will result to the same change as we measured

Next let's unbond half of the tokens we bonded in previous transaction which will halve bonded ratio (approximately):
```
./simd tx staking unbond cosmosvaloper1c6qqnssyraq6f85x77nk5ggjp83ux5z6jhkkc6 500000000000000000stake --from cosmos1c6qqnssyraq6f85x77nk5ggjp83ux5z6hrzr5f
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/e170fa84-e728-4879-8784-71ec3edb19c4)

Again getting current inflation rate

![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/c3966bac-decf-4582-9b42-0ed6f83b0fb9)

The change now is `0.000000020597255542` which is more than after first transaction but less than it was at the beginning as expected

Now let's burn half of tokens on our account which will lead to doubling of bonding ratio return it back to approximately 0.0000001:
```
./simd tx bank burn cosmos1c6qqnssyraq6f85x77nk5ggjp83ux5z6hrzr5f 5000000000000000000000000stake
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/63dc672c-99b5-4729-93d6-b88e4182f875)

Measuring inflation
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/712d8098-8656-4844-a7c1-6a71f1204398)

Here inflation rate change returned back to `0.000000020597254005` as it was after we bonded tokens where we had bonded ratio `~0.0000001` (that's happened because supply halving doubled bonded ratio)

# 3. Change inflation mechanism to be static

Firstly, let's reset data to initial state:
```
./simd comet unsafe-reset-all
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/247776d3-d5e6-429d-92aa-9850b940b298)

Now we'll rebuild SimApp
```
cd ../
make build
```
![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/dd8bdd4d-596c-41d2-b528-73bddf443f5b)

Now we can launch SimApp again

![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/a2645062-6b08-4cf7-b276-e60a75bde9ba)

Let's measure the inflation rate as we did before

![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/416454ce-341a-43dd-8a3c-649913a1dc0c)

Now we can see that it stays on it's initial value - 0.13

Let's do the same transactions as we did before and see if inflation does change

Delegation of `1000000000000000000stake` to validator

![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/2d05869e-73e4-4bbe-adb3-bd6364eb9a6b)

Unbonding `500000000000000000stake` from validator

![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/20031aa3-7c43-4595-a97b-36b8b7e80222)

Burning `5000000000000000000000000stake`

![image](https://github.com/mrrrlen/cosmos-sdk-simapp-inflation/assets/31540338/b73fdbda-0386-4575-bc37-de935fed1035)

So as we can see all those transactions didn't affect the inflation rate change

