# 클라우드에서 스케줄링

분산 프로그램의 효율성은 분산 컴퓨터에서 구성 태스크가 스케줄링되는 방식에 달려있다. 일반적으로 태스크용 클래스와 작업용 클래스의 두 가지 주 클래스로 분류된다. 태스크는 가장 세분화된 실행 단위이며 하나의 작업이 하나 이상의 태스크를 포함할 수 있다. 

심각한 성능 저하를 방지하기 위해 잡 스케줄러는 기본 클라우드 시스템의 이질성을 고려해야 한다. 예를 들어 같은 작업에 속하는 유사한 태스크는 서로 다른 속도의 노드에 있는 이질적 클라우드에서 예약될 수 있는데, 이는 부하 불균형이 발생할 수 있다.

또한 태스크 스케줄러는 시스템 사용률을 향상시키고 작업 병렬 처리를 개선하는 방안을 모색해야 한다. 여기서 목표는 사용가능한 리소스를 공정하게 활용하고 병렬 처리를 효율적으로 늘리는 방식으로 클러스터 컴퓨터 간 태스크를 균등하게 분산시키는 것이지만 이 목표는 다소 모순된 우선 순위를 따른다. 예를 들어 Hadoop 클러스터의 컴퓨터에는 서로 다른 수의 HDFS 블록이 포함될 수 있어, 데이터 로컬리티가 더 높은 경우 해당 컴퓨터의 모든 매핑 태스크가 스케줄링 될 수 있다. 지역성이 해결되면 CPU 사용률이 향상되고, 컴퓨터 간 부하가 분산되며, 태스크 병렬 처리도가 늘어날 수 있다. 

작업 및 태스크를 스케줄링 할 때 다른 과제는 최종 사용자의 성능 기대치를 반영하는 SLO를 충족하는 것이다. 멀티 테넌시의 이기종 클러스터에서 SLO는 다른 작업이 실행되는 동안 새 작업이 도착하는 경우 달성하기가 어렵다. 이 경우 지정된 SLO를 달성하기 위해 실행중인 태스크를 일시 중단하고 새 태스크가 계속 진행하도록 할 수 있는데, 이를 태스크 탄력성(task elasticity)라 한다. 탄력성을 제공하는 일은 어려울 수 있으며, 정확성에 영향을 주지 않으면서 태스크를 일시 중단할 수 있는 안전한 시점을 식별해야 하고, 다시 시작 시 커밋된 작업을 반복하지 않아야 한다. 이는 OS의 context switching과 유사하다.