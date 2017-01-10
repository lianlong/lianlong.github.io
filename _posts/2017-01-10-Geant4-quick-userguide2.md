---
layout: post
title:  "Geant4 入门教程(一)"
categories: Geant4入门
tags:  Geant4入门 
author: lianlong
---

* content
{:toc}


## 质子打靶Microbeam52mm代码说明

###代码结构

如图为代码的结构图，代码中有两个文件夹include和src分别表示头文件的位置和源代码的位置，include文件中用于构建对象和声明的成员函数，src文件夹中对应的文件用于具体实现成员函数的功能，主函数在Microbeam.cc 文件中，run.png是图形化界面的一个图标，CmakeLists.txt 是cmake 所需要的文件，gui.mac、icon.mac、init\_vis.mac、vis.mac 是用于可视化的宏命令文件，Geant4 在主函数中有调用，方便按照用户的需求生成需要的图形化界面，run.mac，run01.mac，run02.mac是直接采用宏命令仿真，不显示图形化界面。

![代码结构图](http://oj9jepyz4.bkt.clouddn.com/src.JPG)






### Geant4的简单介绍

基于Geant4编写单粒子翻转仿真程序，下面几个部分是必须的：

* main()函数<br/>
  在main()中主要是调用相应的管理类初始化内核。G4RunManager是Geant4中最核心的管理类，用于管理整个程序的运行，UNIX系统中还可以构造G4MTRunManager多线程运行Geant4(在window系统中只支持单线程运行)，G4UIManager类用于实现交互命令，实现用户与程序的连接，可以可视化输出数据结果，输出仿真结果等功能。
* 构造必须的用户行为(Action)类<br/>
  有四个用户类需要定义，用户需要继承这几个类建立自己的类，这四个类分别为G4VUserDetectorConstruction、G4VUserPrimaryGeneratorAction类、G4VUserPhysicsList类和G4VUserAction-Initialization,这四个类分别完成了探测器结构的建立、初级入射粒子的性质定义、粒子与物质相互作用的相关物理过程和用户初始化，程序在主函数中开始运行时会首先检测这四个类和类中的一些必要虚函数是否被定义，如果未定义，Geant4内核将会停止运行。
* 构建用户需要的行为类<br/>
 Geant4提供了许多用户类，用户可以按照自己的需要定义行为，在本课题研究中除了必须的类还需要以下几个类需要继承建立自己的类，分别是G4UserRunAction、G4UserEventAction和G4UserSteppingAction。<br/>
 Run事件是Geant4中最大的模拟单元，G4UserRunAction中包含两个虚函数BeginOfRunAction()和EndOfRunAction(),两者分别用于Run开始前初始化和时间结束时统计需要的信息。Event表示一个事件，包含模拟事件中的所有输入输出信息，G4UserEventAction类中的两个虚函数BeingOfEventAction() 和EndOfEventAction()分别用于事件开始前的预备工作和解释时有用信息的提取。
 Step是Geant4中的最小单元，Geant4中以“步”作为最小单位统计各种相关信息，包括Step开始前后的粒子能量、方向等相关信息，然后根据用户需要统计最终需要的信息传递到Event中。

### 主函数Microbeam.cc文件分析

下面为代码中Microbeam.cc的头文件部分，MicrobeamDetectorConstruction.hh为构造探测器对象的头文件，MicrobeamPrimaryGeneratorAction.hh为初级粒子产生器，用于定义初级粒子的相关参数用于 MicrobeamActionInitialization.hh为初始化文件，用于初始化系统出了探测器和物理过程外的其他对象 G4UImanager,G4UIcommand，G4VisExecutive为系统用于UI交互的相关头文件，该部分代码在基本所有的G4 代码中是通用的，不需要有较多深入。G4MTRunManager.hh和G4RunManager.hh用于管理系统整体的运行，系统内部通过该对象对用户编写的代码和系统中原有的代码进行管理控制，实现程序的运行，FTFP\_BERT.hh，QBBC.hh，QGSP\_BIC\_HP.hh等都是Geant4 自带的物理过程，用户可以根据需要选择，或者直接构建自己的物理过程。
Randomize.hh和TRandom3.hh用于产生随机数。<br/>

```javascript
    #include "MicrobeamDetectorConstruction.hh"
    #include "MicrobeamActionInitialization.hh"
    #include <time.h>
    #ifdef G4MULTITHREADED
    #include "G4MTRunManager.hh"
    #else
    #include "G4RunManager.hh"
    #endi
    #include "G4UImanager.hh"
    #include "G4UIcommand.hh"
    #include "FTFP_BERT.hh"
    #include "QBBC.hh"
    #include "QGSP_BIC_HP.hh"
    #include "QGSP_BIC.hh"
    #include "QGSP_BERT_HP.hh"
    #include "QGSP_INCLXX.hh"
    #ifdef G4VIS_USE
    #include "G4VisExecutive.hh"

    #include "MicrobeamPrimaryGeneratorAction.hh"
    #include "Randomize.hh"
```

接下来这部分用于产生名称空间，用户交互接口等，无需关注过多，需要指出的是Geant4 支持多线程运行，其中的变量nTreads表示用户需要运行线程的个数，用户可以根据自己的服务器配置和资源情况设置初始值，这里设置为10.

```javascipt
namespace {
void PrintUsage() {
G4cerr << " Usage: " << G4endl;
G4cerr << " exampleT01 [-m macro ] [-u UIsession] [-t nThreads]" << G4endl;
G4cerr << "   note: -t option is available only for multi-threaded mode."
	   << G4endl;
	  }
}
//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......
    int main(int argc,char** argv)
    {
  // Evaluate arguments
  //
  if ( argc > 7 ) {
    PrintUsage();
    return 1;
    }
    G4String macro;
    G4String session;
    #ifdef G4MULTITHREADED
    G4int nThreads = 10;
    #endif
      for ( G4int i=1; i<argc; i=i+2 ) {
        if      ( G4String(argv[i]) == "-m" ) macro = argv[i+1];
        else if ( G4String(argv[i]) == "-u" ) session = argv[i+1];
    #ifdef G4MULTITHREADED
        else if ( G4String(argv[i]) == "-t" ) {
          nThreads = G4UIcommand::ConvertToInt(argv[i+1]);
        }
    #endif
        else {
          PrintUsage();
          return 1;
        }
      }
      // Choose the Random engine
      //
      G4Random::setTheEngine(new CLHEP::RanecuEngine);
      G4Random::setTheSeed((int)time(0));
      TRandom3 r0(0);
```
该部分代码实现的功能是对Geant4建立运行管理对象，初始化探测器结构，将探测器结构注册到runManager中，建立物理过程对象physics和行为初始化对象actionInitialization，然后将这三个对象注册到runManager中，然后runManager\-\textgreater Initialize()对内核进行初始化，建立探测器结构，通过物理过程搜索相应的数据，并对用户行为进行初始化。runManager\-\textgreater Initialize()后的代码是实现用户命令和内核的交互，对于所有G4 程序基本一样，所以不再分析，最后删除runManager管理类对象，将资源交付给操作系统。本文后续的分析也基本是基于G4的运行过程进行分析。
```javascript
#ifdef G4MULTITHREADED
G4MTRunManager * runManager = new G4MTRunManager;
if ( nThreads > 0 ) {
runManager->SetNumberOfThreads(nThreads);
 }
  #else
  G4RunManager * runManager = new G4RunManager;
  #endif
  // set mandatory initialization classes
  MicrobeamDetectorConstruction* detector = new MicrobeamDetectorConstruction;
  runManager->SetUserInitialization(detector);
//G4VUserPhysicsList* physics = new QBBC;
  G4VUserPhysicsList* physics = new QGSP_BIC;
  //G4VUserPhysicsList* physics = new QGSP_INCLXX;
  //G4VUserPhysicsList* physics = new QGSP_BERT_HP;
  //G4VUserPhysicsList* physics = new FTFP_BERT;
  //G4VUserPhysicsList* physics = new ExN01PhysicsList;
  runManager->SetUserInitialization(physics)

  MicrobeamActionInitialization* actionInitialization
	  =new MicrobeamActionInitialization(detector);
  runManager->SetUserInitialization(actionInitialization);

  // Initialize G4 kernel
  runManager->Initialize();

#ifdef G4VIS_USE
  // Initialize visualization
  G4VisManager* visManager = new G4VisExecutive;
  visManager->Initialize();
#endif
  // Get the pointer to the UI manager and set verbosities
  //
  G4UImanager* UImanager = G4UImanager::GetUIpointer();
  if ( macro.size())
  {
	  //batch mode
	  G4String command = "/control/execute ";
	  UImanager->ApplyCommand(command+macro);
  }
  else
  {
	  //interactive mode: define UI session
#ifdef G4UI_USE
	  G4UIExecutive* ui = new G4UIExecutive(argc, argv, session);
#ifdef G4VIS_USE
	  UImanager->ApplyCommand("/control/execute init_vis.mac");
#endif
	  if (ui->IsGUI())
		UImanager->ApplyCommand("/control/execute gui.mac");
	  ui->SessionStart();
	  delete ui;
#endif
  }
#ifdef G4VIS_USE
  delete visManager;
#endif
  delete runManager;
return 0;
```

### 探测器结构的建立

探测器结构的建模需要进行探测器材料的定义和几何结构的定义,该对象继承自G4VUserDetectorConstruction.hh主要通过 G4VPhysicalVolume* Construct()方法实现探测器的构造。
\par{Geant4}中设计了三个重要的类用于定义材料：G4Isotope、G4Element、G4Material。G4Isotope类用于描述原子本身的性质，比如原子序数、摩尔质量等；G4Element用于描述元素的性质，比如有效核子数、原子丰度等，一般情况下不用G4Isotope定义原子，直接调用G4Element定义元素，采用默认的核素分布，除非要修改元素中各个同位素的相对丰度；G4Material用于描述物质的宏观性质，比如物质的状态、混合物混合情况等。下面代码为典型的几种定义材料的方法：

```javascript

//`真空的定义`
density = universe_mean_density; //from PhysicalConstants.h
pressure = 1.e-5*pascal;
temperature = 300*kelvin;
G4Material* Vacuum = new G4Material(name="Galactic",z=1.,
                  a=1.01*g/mole, density, kStateGas,temperature,pressure)
//W`材料的定义`
G4Material* Wolfram = man->FindOrBuildMaterial("G4_W");
a = 14.01*g/mole;
G4Element* elN  = new G4Element(name="Nitrogen", symbol="N", z=7., a);
G4Element* Si = new G4Element("Silicon",symbol="Si" , z= 14., a= 28.09*g/mole);
G4Element* O  = new G4Element("Oxygen"  ,symbol="O" , z= 8., a= 16.00*g/mole);
//`氧化硅的定义`
G4Material* SiO2 =
new G4Material("quartz", density= 2.200*g/cm3, ncomponents=2);
SiO2->AddElement(Si, natoms=1);
SiO2->AddElement(O, natoms=2)
```

探测器要求定义探测器本身几何、材料、可视化属性及一些用户特定需求。G4采用逻辑体(Logical Volume)的概念来管理这些探测器单元属性的描述，使物理体(Physics Volume) 来管理探测器单元空间位置和它们之间的逻辑关系描述，使用实体(Solids Volume)的概念来管理探测器单元自身的几何描述。Geant4中最大结构是World体，所有其他的几何体都是该``世界子体，Geant4必须先建立World体，然后将其他所有体定义到母体中。对于其他几何体，先要定义一个实体，确定几何形状和大小；再定义逻辑体，为实体添加材料，；最后将这个逻辑体放置在一定的空间坐标中，得到物理体，Geant4中所有的结构都是建立在一个世界体中，以世界的中心为坐标原点，水平向右为y轴正半轴，垂直向上为x轴正半轴,垂直于桌面向上是z轴的正半轴。下面代码为典型的40mm*40mm*52mm的W块在探测器中的定义,首先创建了一个实体Box,注意其中的长宽高都取半长，然后定义了逻辑体WLog，确定了材料为W，最后将该逻辑体放置到坐标(0,0,0)处。它的母体是worldLog，G4PVPlacement中第一个变量表示旋转向量，第二个参数表示中心坐标位置，第三个参数表示其逻辑体，第四个参数表示该实体的名称，可以是任意字符串，第五个参数表示其母体，第六个参数暂时预留,设置为false,第七个参数表示复制次数。

```javascript
G4double w_hx = 40*mm;
G4double w_hy = 52.*mm;
G4double w_hz = 40*mm;
G4Box* WBox = new G4Box("W_box", w_hx/2, w_hy/2, w_hz/2);
WLog = new G4LogicalVolume(WBox, Wolfram, "W_log");
G4VPhysicalVolume* WPhy = new G4PVPlacement(0, G4ThreeVector(0,0,0),
                              WLog, "W", worldLog, false, 0)
```

### 初级粒子产生器行为(MicrobeamPrimaryGeneratorAction.cc)

初级粒子的种类、能量、动量、发散角、能散等参数都是在MicrobeamPrimary\\GeneratorAction.cc文件中定义的，该对象继承自G4VUserPrimaryGeneratorAction.hh,通过GeneratePrimaries方法来实现初级粒子属性的改变。G4是通过G4ParticleGun.hh粒子枪对初级粒子进行管理，所以在MicrobeamPrimaryGeneratorAction的构造函数中首先建立了建立对象G4ParticleGun，然后将相关信息注册到该粒子枪中，实现初级入射粒子的定义。如下代码，表示定义粒子类型为质子。

```javascript
G4ParticleTable* particleTable = G4ParticleTable::GetParticleTable();
G4String particleName;
particleGun->SetParticleDefinition(particleTable->FindParticle                                   (particleName="proton"));
```

接下来依次定义了粒子的入射位置，入射能散，角度散射，最后通过particleGun\-\textgreater GeneratePrimaryVertex(anEvent)注册顶点，该命令必须在所有粒子信息定义后执行。为了能够实现复杂的比如高斯分布等数学函数，在头文件中调用了TMath.h等相关文件。

```javascipt
G4double sigma = 2;
TRandom3 r(0);
G4double beamRadius = 2*mm;
G4double x = r.Gaus(0,beamRadius/sigma);
G4double y = fDetConstruction->fSourceY;
G4double z = r.Gaus(0,beamRadius/sigma);
while(x*x+z*z>=beamRadius*beamRadius)
{
x = r.Gaus(0,beamRadius/sigma);
z = r.Gaus(0,beamRadius/sigma);
}
particleGun->SetParticlePosition(G4ThreeVector(x,y,z));
G4double energySpd = 1E-3;
G4double energycenter = 300*MeV;
G4double energy = r.Gaus(energycenter, energySpd/sigma*energycenter);
particleGun->SetParticleEnergy(energy);
G4ThreeVector v(0,1,0);
G4double maxtheta = 0.005;
G4double theta = r.Gaus(0,maxtheta/sigma);
while(TMath::Abs(theta)>=maxtheta)
theta = r.Gaus(0,maxtheta/sigma);
G4double phi = 2*TMath::Pi()*r.Rndm();
v.setRThetaPhi(1,theta,phi);
v.rotateX(-90*degree);
particleGun->SetParticleMomentumDirection(v);

particleGun->GeneratePrimaryVertex(anEvent);
```

### 用户定义类型RunAction(MicrobeamRunAction.cc)

在Geant4中，Run是一个最大的模拟单位，一个run由一些列事件(event)组成，它是G4Run的一个对象，有G4RunManager的方法beamOn()启动，用户可以在此基础上构建用户自定义run行为，MicrobeamRunAction.cc继承自G4UserRunAction，这个基类有两个虚拟方法beginOfRunAction()和endOfRunAction(),前者在beamOn调用之前用于初始化数据，登记root统计图等任务，后者在beamOn运行结束后调用，用于存储或者输出统计结果。考虑到质子入射W打靶试验中需要统计各个截面的数据，所以利用C++泛函数编程的方法定义如下map作为成员用于统计粒子类型和粒子数目：

```javascirpt
std::map<G4String, G4double> particleList;
std::map<G4String, G4int> particleListN
```

下面代码是MicrobeamRunAction构造函数的一部分},G4AnalysisManager用于构建root软件分析管理对象，analysisManager\-\textgreater SetVerboseLevel(1)表示显示精度的级别,后面几行用于创建元组，建立root管理的表头。然后在beginOfRunAction定义了需要输出的文件，在EndOfRunAction方法中，将需要的数据存储到文件中，并关闭了root等文件，  G4AnalysisManager* analysisManager = G4AnalysisManager::Instance()用于找到分析管理对象指针，用于进一步操作。

```javascrpit
 G4AnalysisManager* analysisManager = G4AnalysisManager::Instance();
  G4cout << "Using " << analysisManager->GetType() << G4endl;
  // Create directories
  analysisManager->SetVerboseLevel(1);
  // Creating ntuple
  analysisManager->CreateNtuple("Vertex", "Vertex Info");
  analysisManager->CreateNtupleDColumn(0,"x");
  analysisManager->CreateNtupleDColumn(0,"y");
  analysisManager->CreateNtupleDColumn(0,"z");
  analysisManager->CreateNtupleDColumn(0,"Theta");
  analysisManager->CreateNtupleDColumn(0,"p");
  analysisManager->CreateNtupleDColumn(0,"E");
  analysisManager->FinishNtuple(0);
```

### 用户定义类型事件EventAction(MicrobeamEventAction.cc)

事件是一个包含所有被模拟事件的输入输出信息的类的实例，通俗的来讲，一个粒子从入射到相关所有的反应结束，即为一个事件，事件用于统计一个粒子入射后的相关反应情况，可以统计初级粒子属性，或者通过step中传递过来的参数，累加一个事件中总的能量变化等功能。在本代码中主要是在事件结束后完成了初级粒子相关信息的统计，主要涉及到统计中Ntuple的使用，其中前三行代码分别获得分析管理类，粒子顶点，初级粒子对象指针，方便调用其方法，然后通过FillNtupleDColumn方法将数据填充到Ntuple中，第一参数0表示添加到元表的第一行中，第二个参数表示添加Ntuple对应的列中，第三个参数表示输入相应的值，这部分具体可以查看生成的root文件，可以很明了的看到相应的区别，这里统计了初级粒子的X、Y、Z轴坐标，散射角度、总的动量和动能。

```javascript
G4AnalysisManager* analysisManager = G4AnalysisManager::Instance();	
G4PrimaryVertex* gpVertex = event->GetPrimaryVertex();
G4PrimaryParticle* gpParticle = gpVertex->GetPrimary();
G4ThreeVector pd =  gpParticle->GetMomentumDirection();
G4double theta = acos(pd.getY()/pd.getR());
// fill ntuple
analysisManager->FillNtupleDColumn(0, 0, gpVertex->GetX0()/mm);
analysisManager->FillNtupleDColumn(0, 1, gpVertex->GetY0()/mm);
analysisManager->FillNtupleDColumn(0, 2, gpVertex->GetZ0()/mm);
analysisManager->FillNtupleDColumn(0, 3, theta);
analysisManager->FillNtupleDColumn(0, 4, gpParticle->GetTotalMomentum()/MeV);
analysisManager->FillNtupleDColumn(0, 5, gpParticle->GetKineticEnergy()/MeV);
analysisManager->AddNtupleRow(0);
```

### 用户自定义类型StepAction(MicrobeamSteppingAction.cc)

step是Geant4}中最小的仿真单位，Geant4以step作为基本单位，统计每个step开始和结束的相关信息，获得相应的数据，再决定下一步如何仿真，可以统计每个step之前和之后所在的逻辑体，能量，动量，粒子质量等相关信息。用户可以从G4UserSteppingAction中派生出用户自定义行为，实现需要的功能，G4UserSteppingAction中包含虚函数UserSteppingAction，用户可以在该函数中定义自己需要实现的功能。<br/>
如下代码表示获得当前step开始点和结束点的物理体、当前step的轨迹对象指针(track是比step稍微打一点的仿真单位，多个step构成一个track)、总能量、动能、位置、粒子名称、动量方向。

```javascript
 G4VPhysicalVolume* pre_volume
    = step->GetPreStepPoint()->GetTouchableHandle()->GetVolume();
  G4VPhysicalVolume* post_volume
    = step->GetPostStepPoint()->GetTouchableHandle()->GetVolume();
  G4Track* track = step->GetTrack();
  // energy deposit
  G4double edep = step->GetTotalEnergyDeposit();
  G4ThreeVector p = step->GetPreStepPoint()->GetPosition();
  G4double e=step->GetPreStepPoint()->GetKineticEnergy();
  G4String name = step->GetTrack()->GetDefinition()->GetParticleName();
  G4double x=p.getX();
  G4double y=p.getY();
  G4double z=p.getZ();
  G4ThreeVector pd = step->GetPreStepPoint()->GetMomentumDirection();
  G4double theta = acos(pd.getY()/pd.getR())
```

接下来这段代码用于判断当前step前一个物理体是A，后一个物理体是W时，执行if中的语句，即判断通过A界面的step，然后统计此时粒子的相关信息，输入到root文件中，同时对runAction中构建的统计粒子类型和数目的map进行累加.

```javascript
G4AnalysisManager* analysisManager = G4AnalysisManager::Instance();
G4int trackID = track->GetTrackID();
if(pre_volume == fDetConstruction->GetVolume('A')
    && post_volume == fDetConstruction->GetVolume('W'))
{
	G4double mass = step->GetPreStepPoint()->GetMass();
	G4double energy = step->GetPreStepPoint()->GetKineticEnergy();
	G4String pname = track->GetParticleDefinition()->GetParticleName();
	if((mass==0 && pname=="gamma")||(mass!=0))
	{
	analysisManager->FillNtupleDColumn(2, 0, x/mm);
	analysisManager->FillNtupleDColumn(2, 1, y/mm);
	analysisManager->FillNtupleDColumn(2, 2, z/mm);
	analysisManager->FillNtupleDColumn(2, 3, theta);
	analysisManager->FillNtupleDColumn(2, 4, mass/MeV);
	analysisManager->FillNtupleDColumn(2, 5, energy/MeV);
	analysisManager->FillNtupleIColumn(2, 6, trackID);
	analysisManager->AddNtupleRow(2);
    }
}
```

###其他用户行为

除了上述Action外，还有G4StackingAction, G4MagneticField, G4TrackActioin 等用户行为，G4StackingAction是定义栈行为，通过栈决定下一时刻当前粒子进行的操作，G4MagneticField用于定义磁场，G4TrackAction用于定义Track行为。用户可根据自己的需要，通过派生自己的类，构建虚函数，定义自己的用户行为。

## 其他问题

### Geant4单位系统

{Geant4}的单位是统一的，基于HepSystemOfUnits,可以在G4SystemOfUnits.hh中找到定义，另外Geant4对各个变量有默认的单位，如果不写单位的话按照默认单位定义。
* 长度 mm
* 能量 MeV
* 质量 Kg
* 时间 ns

### root的使用

1. had  \- f0 Micro.root Micro\_ t* 表示将各个进程中的所有root文件合并到一个中
2. 终端下输入root打开root软件
3. 在root中输入TBrowser browser可以打开GUI界面，然后点击相应Micro.root文件可以得到输出结果如图：
![root gui界面](http://oj9jepyz4.bkt.clouddn.com/rootGUI.jpg)
![root 统计桌面](http://oj9jepyz4.bkt.clouddn.com/data.jpg)

## 说明
上述该教程比较简单，适合作为入门者大致了解Geant4代码，如有不足之处敬请原谅。