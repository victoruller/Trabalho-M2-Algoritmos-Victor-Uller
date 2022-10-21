// Trabalho-M2-Algoritmos-Victor-Uller
#include <iostream>
#include <fstream>
#include <conio.h> //getch()
#include <windows.h> ///remover
#include <ctime>

using namespace std;

struct PLAYER {
    //cria tres dimensões para os players para terem mais mapas nas camadas "z"
    int x,y,z;
    float *top;
    bool pause=0, ganhou=0, t=1;


    void criatop(int a){
    top = new float[a];
    }

    void coord (int li, int co, int pr){
        x=li;
        y=co;
        z=pr;
    }
};


struct ARQUIVO {
    char* nome;
    int x, y, z;
    int *** m;


    void tam(int li, int co, int pr){
        x=li;
        y=co;
        z=pr;
    }


    void carrega (PLAYER &p, PLAYER &p2, bool t){
      //"t" faz com que a criação da matriz e ranking acontessam apenas ao iniciar o jogo pela primeira vez
      if(t){
        //gera uma matriz sólida para armazenar também o que tem além dos portais
         m = new int**[x];
        for (int i=0; i<x; i++){
            m[i]=new int*[y];
            for (int j=0; j<y; j++){
               m[i][j]=new int[z];
            }
        }
      }
        ifstream mapa;
        mapa.open(nome);
        char c;

    mapa >> p.x;
    mapa >> p.y;
    mapa >> p.z;
    mapa >> p2.x;
    mapa >> p2.y;
    mapa >> p2.z;

        ///coloca na matriz o mapa
        for(int i=0; i<x;i++){
            for(int j=0; j<y;j++){
                mapa>>c;
                m[i][j][0]=c-48;
            }
        }

    ///coloca na camada z1 o mapa 2
        for(int i=0; i<30;i++){
            for(int j=0; j<30;j++){
                    mapa>>c;
                    m[i][j][1]=c-48;
            }
        }

     if(t){
      p.criatop(5);
      for(int o=0; o<5; o++){
        mapa >> p.top[o];
      }
    }
        mapa.close();
    }


};


void mostrarCursor(bool showFlag){
    HANDLE out = GetStdHandle(STD_OUTPUT_HANDLE);

    CONSOLE_CURSOR_INFO     cursorInfo;

    GetConsoleCursorInfo(out, &cursorInfo);
    cursorInfo.bVisible = showFlag; // set the cursor visibility
    SetConsoleCursorInfo(out, &cursorInfo);
}


void colorir (int cor) {

    HANDLE out = GetStdHandle(STD_OUTPUT_HANDLE);
    SetConsoleTextAttribute(out, cor);
}


void posicaoxy( int column, int line ){
        COORD coord;
        coord.X = column;
        coord.Y = line;
        SetConsoleCursorPosition(GetStdHandle( STD_OUTPUT_HANDLE ),coord);
    }


void relogio(clock_t inicio){

    //posicaoxy(0, 20);

    //cout<<"                              ";

    posicaoxy(0, 15);

    cout<<"relogio: "<<( clock() - inicio ) / (double) CLOCKS_PER_SEC;

}

void mostra_mapa(ARQUIVO a, PLAYER &p) {
    int v=3;
    posicaoxy(0,0);
    for (int i=p.x-v; i<=p.x+v; i++){
        for (int j=p.y-v; j<=p.y+v; j++){
                //não deixa ler o que está fora do mapa
            if(i > -1 and j > -1 and i < a.x and j < a.y){

              //quando vê o player 2 na tela ganha, ja que ele é um dígito 2.
              if(a.m[i][j][p.z]==2){
                //checa se o 2 visto não é o próprio player 1.
                if(i!=p.x or j!=p.y){
                p.ganhou = 1;
                }}
              switch(a.m[i][j][p.z]){
              case 0: cout<<" "; break;
              case 1: cout<<char(219); break;
              case 2: cout<<char(1); break;
              case 3: cout<<'8'; break;
              }} else{
            cout<<"/";
            }
        }
        cout<<"\n";
    }

}

void mostrar_mapa2(ARQUIVO a, PLAYER p2){
    int v=3;
    posicaoxy(0,20);

    //variável xx serve para a saída com o mapa não voltar à coluna 1 a cada linha;
    for (int i=p2.y-v, xx=0; i<=p2.y+v; i++, xx++){
            posicaoxy(30,xx);
        for (int j=p2.x-v; j<=p2.x+v; j++){
            if(i > -1 and j > -1 and i < a.y and j < a.x){
            switch(a.m[j][i][p2.z]){
            case 0: cout<<" "; break;
            case 1: cout<<char(219); break;
            case 2: cout<<char(1); break;
            case 3: cout<<'8'; break;
            }} else{
            cout<<"/";
            }
        }
        cout<<"\n";
    }
    posicaoxy(45,0);
    cout << "T = Pause";
}

char getch_v2(){
    if (kbhit()){
        return getch();
    }
    return -1;
}

void mover (PLAYER &p, PLAYER &p2, ARQUIVO &mapa){
    char tecla=getch_v2();
    switch (tecla){
    case 'w':
        switch(mapa.m[p.x-1][p.y][p.z]){
          case 0:
            mapa.m[p.x][p.y][p.z]=0;
            mapa.m[p.x-1][p.y][p.z]=2;
            p.x=p.x-1;
            break;
          case 3:
            mapa.m[p.x][p.y][p.z]=0;
            mapa.m[p.x-1][p.y][p.z]=3;
            //se z=1, (1+1)%2 = 0, e se z=0, (0+1)%2 = 1
            p.z=(p.z+1)%2;
            break;
        }
        break;
    case 's':
          switch(mapa.m[p.x+1][p.y][p.z]){
          case 0:
            mapa.m[p.x][p.y][p.z]=0;
            mapa.m[p.x+1][p.y][p.z]=2;
            p.x=p.x+1;
            break;
          case 3:
            mapa.m[p.x][p.y][p.z]=0;
            mapa.m[p.x+1][p.y][p.z]=3;
            p.z=(p.z+1)%2;
            break;
        }
        break;
    case 'a':
          switch(mapa.m[p.x][p.y-1][p.z]){
          case 0:
            mapa.m[p.x][p.y][p.z]=0;
            mapa.m[p.x][p.y-1][p.z]=2;
            p.y=p.y-1;
            break;
          case 3:
            mapa.m[p.x][p.y][p.z]=0;
            mapa.m[p.x][p.y-1][p.z]=3;
            p.z=(p.z+1)%2;
            break;
        }
        break;
    case 'd':
          switch(mapa.m[p.x][p.y+1][p.z]){
          case 0:
            mapa.m[p.x][p.y][p.z]=0;
            mapa.m[p.x][p.y+1][p.z]=2;
            p.y=p.y+1;
            break;
          case 3:
            mapa.m[p.x][p.y][p.z]=0;
            mapa.m[p.x][p.y+1][p.z]=3;
            p.z=(p.z+1)%2;
            break;
        }
        break;
    case 'j':
        switch(mapa.m[p2.x-1][p2.y][p2.z]){
          case 0:
            mapa.m[p2.x][p2.y][p2.z]=0;
            mapa.m[p2.x-1][p2.y][p2.z]=2;
            p2.x=p2.x-1;
            break;
          case 3:
            mapa.m[p2.x][p2.y][p2.z]=0;
            mapa.m[p2.x-1][p2.y][p2.z]=3;
            p2.z=(p2.z+1)%2;
            break;
        }
        break;
    case 'l':
         switch(mapa.m[p2.x+1][p2.y][p2.z]){
          case 0:
            mapa.m[p2.x][p2.y][p2.z]=0;
            mapa.m[p2.x+1][p2.y][p2.z]=2;
            p2.x=p2.x+1;
            break;
          case 3:
            mapa.m[p2.x][p2.y][p2.z]=0;
            mapa.m[p2.x+1][p2.y][p2.z]=3;
            p2.z=(p2.z+1)%2;
            break;
        }
        break;
    case 'i':
        switch(mapa.m[p2.x][p2.y-1][p2.z]){
          case 0:
            mapa.m[p2.x][p2.y][p2.z]=0;
            mapa.m[p2.x][p2.y-1][p2.z]=2;
            p2.y=p2.y-1;
            break;
          case 3:
            mapa.m[p2.x][p2.y][p2.z]=0;
            mapa.m[p2.x][p2.y-1][p2.z]=3;
            p2.z=(p2.z+1)%2;
            break;
        }
        break;
    case 'k':
        switch(mapa.m[p2.x][p2.y+1][p2.z]){
          case 0:
            mapa.m[p2.x][p2.y][p2.z]=0;
            mapa.m[p2.x][p2.y+1][p2.z]=2;
            p2.y=p2.y+1;
            break;
          case 3:
            mapa.m[p2.x][p2.y][p2.z]=0;
            mapa.m[p2.x][p2.y+1][p2.z]=3;
            p2.z=(p2.z+1)%2;
            break;
        }
        break;
    case 't':
        p.pause = 1;
        break;
    default:
        break;
    }

}


void salvar(PLAYER p, PLAYER p2, ARQUIVO mapa){
ofstream save;
save.open("saved.txt");

save << p.x << "\n" << p.y << "\n" << p.z << "\n" << p2.x << "\n" << p2.y << "\n" << p2.z << "\n";

for(int i=0; i<mapa.x; i++){
  for(int j=0; j<mapa.y; j++){
    save << mapa.m[i][j][0];
  }  save << "\n";
}

for(int i=0; i<30; i++){
  for(int j=0; j<30; j++){
    save << mapa.m[i][j][1];
  }  save << "\n";
}

for(int i=0; i<5; i++){
        save << p.top[i] << "\n";
    }


save.close();
}

void novojogo(PLAYER &p, PLAYER &p2, ARQUIVO &mapa, bool t){
    mapa.nome="labirinto.txt";
    mapa.tam(41,40,2);
    mapa.carrega(p, p2, t);
}

void velhojogo(PLAYER &p, PLAYER &p2, ARQUIVO &mapa){
mapa.nome="saved.txt";
    mapa.tam(41,40,2);
    mapa.carrega(p, p2, 1);
}


//faz uma recursividade para passar para baixo os rankins que serão substituidos sem excluir o de baixo
void arrumarank(PLAYER &p, float tempo, int v){
//para de analizar quando sai do último rank
if(v >= 5){
    p.top[5] = 0.0;
    return;
}
//se o tempo for menor ele chama novamente a função, porém, com o tempo sendo o rank ultrapassado e adiciona o tempo antigo à esse rank.
if(tempo < p.top[v] or p.top[v] == 0.0){
    arrumarank(p, p.top[v], v+1);
    p.top[v] = tempo;
    return;
} else {
//caso não seja menor, ele checa com o próximo colocado
arrumarank(p, tempo, v+1);
return;
}
}



void jogar(PLAYER &p, PLAYER &p2, ARQUIVO &mapa){
        clock_t inicio;

    inicio = clock(); // tempo inicial
while (true){
        relogio(inicio);
        mostra_mapa(mapa, p);
        mostrar_mapa2(mapa, p2);
        mover(p, p2, mapa);
        if(p.pause){
            return;
        }
        if(p.ganhou){
            float tempo = (clock() - inicio) / 1000.00;
            arrumarank(p, tempo, 0);

            system("cls");
            cout << "Player 1 ganhouuu!!!!\n\n";
            cout << "tempo: " << tempo;
            for(int i=0; i<5; i++){
                cout << "\n" << i+1 << " > " << p.top[i];
            }
            cout << "\n\n Aperte t";
           //exige que t seja apertado
            char a='a';
            while(a!='t'){
            a=getch();
           }
           novojogo(p, p2, mapa, false);
            return;
        }
    }
}


void menu(PLAYER &p, PLAYER &p2, ARQUIVO &mapa){
    system("cls");
    p.pause=0;
    p.ganhou=0;
cout << "JOGO\n\n\n\
1 >> Novo Jogo\n\n\
2 >> Continuar\n\n\
3 >> Continuar save\n\n\
4 >> Salvar\n";
char tecla=getch();
system("cls");
    switch (tecla){
    case '1':
        novojogo(p, p2, mapa, 0);
        jogar(p, p2, mapa);
        break;
    case '2':
            jogar(p, p2, mapa);
        break;
    case '3':
        if(!p.pause){
        velhojogo(p, p2, mapa);
        jogar(p, p2, mapa);
        }
        break;
    case '4':
        salvar(p, p2, mapa);
        break;
        }

}


int main()
{




    bool playing=true;


    mostrarCursor(false);
    PLAYER p;
    PLAYER p2;
    ARQUIVO mapa;
    novojogo(p, p2, mapa, true);



    while(playing){
    menu(p, p2, mapa);
    }

    return 0;
}
