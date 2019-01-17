#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>

using namespace std;

#define _CRT_SECURE_NO_WARNINGS
#pragma warning(disable:4996)

int nFiled[21][21]; 
int nVisit[21][21]; // bfs 방문 배열
int nN;
int nRet;
int nShark_Size;

int nDirX[4] = { 0, 0, 1, -1 }; // 아래, 위, 오른, 왼
int nDirY[4] = { 1, -1, 0, 0 };

typedef struct Pos
{
	int nX;
	int nY;
};

typedef struct Min_Info
{
	int nX;
	int nY;
	int nDis;
};

Pos Shark_Pos;
vector <Min_Info> min_dis_vec;

int fnCompare_XY(Min_Info data_1, Min_Info data_2) // 같은 거리의 물고기가 여러마리 일때 정렬조건
{
	if (data_1.nY == data_2.nY)
		return data_1.nX < data_2.nX;
	else
		return data_1.nY < data_2.nY;
}

int bfs(int nCrtX, int nCrtY)
{
	Min_Info Min_Tmp = { nCrtX, nCrtY, 0 };
	
	int nNextX = 0;
	int nNextY = 0;
	int nNextDis = 0;
	int nBfs_Min_Dir = 10000; // 최소의 거리의 물고기만 vector에 담기위해.

	queue <Min_Info> bfs_queue;

	bfs_queue.push(Min_Tmp);

	nVisit[Min_Tmp.nY][Min_Tmp.nX] = 1;

	while (!bfs_queue.empty())
	{
		Min_Tmp = bfs_queue.front();
		bfs_queue.pop();

		for (int i = 0; i < 4; i++)
		{
			nNextX = Min_Tmp.nX + nDirX[i];
			nNextY = Min_Tmp.nY + nDirY[i];
			nNextDis = Min_Tmp.nDis + 1;

			if (nBfs_Min_Dir < nNextDis) // 최소거리의 물고기가 존재한다면 그보다 큰 거리는 탐색하지 않는다.
				break;
			if (nVisit[nNextY][nNextX] == 1)
				continue;

			if (nNextX >= 0 && nNextX < nN && nNextY >= 0 && nNextY < nN && nFiled[nNextY][nNextX] <= nShark_Size)
			{
				bfs_queue.push({ nNextX, nNextY, nNextDis });
				nVisit[nNextY][nNextX] = 1;
				
				if (nFiled[nNextY][nNextX] > 0 && nFiled[nNextY][nNextX] < nShark_Size) // 최소거리의 물고기 담기위한 조건.
				{
					min_dis_vec.push_back({ nNextX, nNextY, nNextDis }); //먹을 수 있는 물고기를 담는 배열.
					if (nBfs_Min_Dir > nNextDis)
						nBfs_Min_Dir = nNextDis;
				}
			}
		}
	}

	return 0;
}

int fnSol()
{
	int nShark_HP = 0;
	int nChk = 0;

	while (true)
	{
		for (int i = 0; i < nN; i++)
			for (int j = 0; j < nN; j++)
				nVisit[i][j] = 0; 

		min_dis_vec.clear();
		bfs(Shark_Pos.nX, Shark_Pos.nY); //min_dis_vec 배열에 먹을 수 있는 물고기를 넣어둠.

		if (min_dis_vec.empty())
			return 0;

		if (min_dis_vec.size() > 1)
		{
			///고기가 여러마리 일때 순서 정하고 먹자.
			sort(min_dis_vec.begin(), min_dis_vec.end(), fnCompare_XY); 
			
			nFiled[Shark_Pos.nY][Shark_Pos.nX] = 0;
			Shark_Pos.nX = min_dis_vec[0].nX;
			Shark_Pos.nY = min_dis_vec[0].nY;
			nShark_HP++;

			if (nShark_HP == nShark_Size)
			{
				nShark_Size++;
				nShark_HP = 0;
			}
			nRet += min_dis_vec[0].nDis;
			nFiled[Shark_Pos.nY][Shark_Pos.nX] = 9;
		}
		else // 먹을 물고기 하나일때.
		{
			nFiled[Shark_Pos.nY][Shark_Pos.nX] = 0;
			Shark_Pos.nX = min_dis_vec[0].nX;
			Shark_Pos.nY = min_dis_vec[0].nY;
			nShark_HP++;

			if (nShark_HP == nShark_Size)
			{
				nShark_Size++;
				nShark_HP = 0;
			}
			nRet += min_dis_vec[0].nDis;
			nFiled[Shark_Pos.nY][Shark_Pos.nX] = 9;
		}
	}
}

int main()
{
	int nT;
	int ntest_case;

	freopen("input.txt", "r", stdin);

	cin >> nT;

	for (ntest_case = 0; ntest_case < nT; ntest_case++)
	{
		cin >> nN;
		for (int i = 0; i < nN; i++)
		{
			for (int j = 0; j < nN; j++)
			{
				cin >> nFiled[i][j];
				nVisit[i][j] = 0;
				if (nFiled[i][j] == 9)
				{
					Shark_Pos.nX = j;
					Shark_Pos.nY = i;
				}
			}
		}
		nShark_Size = 2;
		nRet = 0;

		fnSol();

		cout << "#" << ntest_case << " " << nRet << endl;
	}
	return 0;
}

