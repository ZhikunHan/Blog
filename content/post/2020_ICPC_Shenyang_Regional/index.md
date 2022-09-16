+++
title = "ICPC 2020 Shenyang Regional"
date = "2021-09-06T20:35:24+08:00"
tags = ["算法竞赛"]
categories = ["XCPC"]
math = true
description = "打铁实录"
image = "ICPC_Challenge.png"
+++

CF Gym : [Dashboard - The 2020 ICPC Asia Shenyang Regional Programming Contest - Codeforces](https://codeforces.com/gym/103202)


## D [Journey to Un'Goro](https://codeforces.com/gym/103202/problem/D)

```cpp
#include<bits/stdc++.h>
//#include<bits/extc++.h>
//#define int long long//__int128
#define mmst0(x) memset(x,0,sizeof(x))
#define mmst3f(x) memset(x,0x3f,sizeof(x))
#define pb(x) emplace_back(x)
#define mkp(x, y) make_pair(x, y)
#define fi first
#define se second
using namespace std;
//using namespace __gnu_pbds; //If using pbds don't using std!
typedef long long ll;
typedef long double rld;
typedef unsigned long long ull;

const rld eps = 1e-6;
const int INF=0x3f3f3f3f;//0x3f3f3f3f3f3f3f3f;//LLINF
const int MAXN=(int)3e5+3;

inline char nc(){static char buf[100000],*p1=buf,*p2=buf;return p1==p2&&(p2=(p1=buf)+fread(buf,1,100000,stdin),p1==p2)?EOF:*p1++;}
inline int read(){int s=0,w=1;char ch=nc();while(!isdigit(ch)){if(ch=='-')w=-1;ch=nc();}while(isdigit(ch)){s=(s<<3)+(s<<1)+(ch^48);ch=nc();} return s*w;}
//inline void read(int &x){char ch=nc();x=0;while (!(ch>='0'&&ch<='9')) ch=nc();while (ch>='0'&&ch<='9') x=(x<<3)+(x<<1)+ch-48,ch=nc();}//根据参数个数自动选择
//void prt(int x){if(x<0){putchar('-');x=-x;}if(x>9)prt(x/10);putchar((char)(x%10+'0'));}

int n,cnt;
char s[MAXN];

void dfs(int now, int ji, int ou, bool flag)
{
    if(cnt==100) exit(0);
    if(n%2)
    {
        if(ji+(n-ji-ou) < (n>>1) || ou + (n-ji-ou) < (n>>1) || ji > (n>>1)+1 || ou > (n>>1)+1) return;
    }
    else
    {
        if(ji+(n-ji-ou) < (n>>1) || ou + (n-ji-ou) < (n>>1) || ji > (n>>1) || ou > (n>>1)) return;
    }
    if(now == n)
    {
        for(int i=1;i<=n-1;i++) printf("%c",s[i]);
        printf("\n");
        cnt++;
        return;
    }
    s[now]='b';
    if(flag) dfs(now+1,ji+1,ou,flag);
    else dfs(now+1,ji,ou+1,flag);
    s[now]='r'; //flag取反
    if(!flag) dfs(now+1,ji+1,ou,!flag);
    else dfs(now+1,ji,ou+1,!flag);
}

inline void work()
{
    n=read()+1; ll ans=((ll)n/2) * ((ll)n-(ll)n/2);
    printf("%lld\n",ans);
    dfs(1,0,1,false);
    return;
}

signed main()
{
    //ios::sync_with_stdio(false); cin.tie(nullptr); cout.tie(nullptr); //freopen(".in", "r", stdin);//freopen(".out", "w", stdout);
    signed T=1;//(signed)read();//scanf("%d",&T);//cin>>T;
    for(signed Case=1; Case<=T; Case++)
    {
        //printf("Case %d: ",Case);
        //while(cin>>n) work(n);
        work();
    }
    return 0;
}
//构造一个长度为 n 的只含有 b,r 的字符串,使得子串中 r 数量为奇数的最多
//r 看成 1,b 看成 0 的话,前缀和中为奇数的子串就满足要求
```



## F [Kobolds and Catacombs](https://codeforces.com/gym/103202/problem/F)

```cpp
#include<bits/stdc++.h>
//#include<bits/extc++.h>
//#define int long long//__int128
#define mmst0(x) memset(x,0,sizeof(x))
#define mmst3f(x) memset(x,0x3f,sizeof(x))
#define pb(x) emplace_back(x)
#define mkp(x, y) make_pair(x, y)
#define fi first
#define se second
using namespace std;
//using namespace __gnu_pbds; //If using pbds don't using std!
typedef long long ll;
typedef long double rld;
typedef unsigned long long ull;

const rld eps = 1e-6;
const int INF=0x3f3f3f3f;//0x3f3f3f3f3f3f3f3f;//LLINF
const int MAXN=(int)1e6+3;

inline char nc(){static char buf[100000],*p1=buf,*p2=buf;return p1==p2&&(p2=(p1=buf)+fread(buf,1,100000,stdin),p1==p2)?EOF:*p1++;}
inline int read(){int s=0,w=1;char ch=nc();while(!isdigit(ch)){if(ch=='-')w=-1;ch=nc();}while(isdigit(ch)){s=(s<<3)+(s<<1)+(ch^48);ch=nc();} return s*w;}
//inline void read(int &x){char ch=nc();x=0;while (!(ch>='0'&&ch<='9')) ch=nc();while (ch>='0'&&ch<='9') x=(x<<3)+(x<<1)+ch-48,ch=nc();}//根据参数个数自动选择
//void prt(int x){if(x<0){putchar('-');x=-x;}if(x>9)prt(x/10);putchar((char)(x%10+'0'));}

int n;
int a[MAXN],b[MAXN];

inline void work()
{
    n=read();
    for(int i=1;i<=n;i++) b[i]=a[i]=read();
    sort(b+1,b+1+n,[](int x,int y){return y>x;});
    //for(int i=1;i<=n;i++) printf("%d ",b[i]);
    int cura=0,curb=0,ans=0;
    for(int i=n;i>=1;i--)
    {
        cura+=a[i]; curb+=b[i];
        if(cura==curb) ans++;
    }
    printf("%d\n",ans);
    return;
}

signed main()
{
    //ios::sync_with_stdio(false); cin.tie(nullptr); cout.tie(nullptr); //freopen(".in", "r", stdin);//freopen(".out", "w", stdout);
    signed T=1;//(signed)read();//scanf("%d",&T);//cin>>T;
    for(signed Case=1; Case<=T; Case++)
    {
        //printf("Case %d: ",Case);
        //while(cin>>n) work(n);
        work();
    }
    return 0;
}
```



## G [The Witchwood](https://codeforces.com/gym/103202/problem/G)

```cpp
#include<bits/stdc++.h>
//#include<bits/extc++.h>
#define int long long//__int128
#define mmst0(x) memset(x,0,sizeof(x))
#define mmst3f(x) memset(x,0x3f,sizeof(x))
#define pb(x) emplace_back(x)
#define mkp(x, y) make_pair(x, y)
#define fi first
#define se second
using namespace std;
//using namespace __gnu_pbds; //If using pbds don't using std!
typedef long long ll;
typedef long double rld;
typedef unsigned long long ull;

const rld eps = 1e-6;
const int INF=0x3f3f3f3f;//0x3f3f3f3f3f3f3f3f;//LLINF
const int MAXN=(int)1e3+3;

inline char nc(){static char buf[100000],*p1=buf,*p2=buf;return p1==p2&&(p2=(p1=buf)+fread(buf,1,100000,stdin),p1==p2)?EOF:*p1++;}
inline int read(){int s=0,w=1;char ch=nc();while(!isdigit(ch)){if(ch=='-')w=-1;ch=nc();}while(isdigit(ch)){s=(s<<3)+(s<<1)+(ch^48);ch=nc();} return s*w;}
//inline void read(int &x){char ch=nc();x=0;while (!(ch>='0'&&ch<='9')) ch=nc();while (ch>='0'&&ch<='9') x=(x<<3)+(x<<1)+ch-48,ch=nc();}//根据参数个数自动选择
//void prt(int x){if(x<0){putchar('-');x=-x;}if(x>9)prt(x/10);putchar((char)(x%10+'0'));}

int n,k;
int a[MAXN];

inline void work()
{
    n=read(); k=read();
    for(int i=1;i<=n;i++) a[i]=read();
    sort(a+1,a+1+n,[](int x,int y){return y<x;});
    int ans=0;
    for(int i=1;i<=k;i++) ans+=a[i];
    printf("%lld\n",ans);
    return;
}

signed main()
{
    //ios::sync_with_stdio(false); cin.tie(nullptr); cout.tie(nullptr); //freopen(".in", "r", stdin);//freopen(".out", "w", stdout);
    signed T=1;//(signed)read();//scanf("%d",&T);//cin>>T;
    for(signed Case=1; Case<=T; Case++)
    {
        //printf("Case %d: ",Case);
        //while(cin>>n) work(n);
        work();
    }
    return 0;
}
```



## I [Rise of Shadows](https://codeforces.com/gym/103202/problem/I)

```cpp
#include<bits/stdc++.h>
//#include<bits/extc++.h>
#define int long long//__int128
#define mmst0(x) memset(x,0,sizeof(x))
#define mmst3f(x) memset(x,0x3f,sizeof(x))
#define pb(x) emplace_back(x)
#define mkp(x, y) make_pair(x, y)
#define fi first
#define se second
using namespace std;
//using namespace __gnu_pbds; //If using pbds don't using std!
typedef long long ll;
typedef long double rld;
typedef unsigned long long ull;

const rld eps = 1e-6;
const int INF=0x3f3f3f3f;//0x3f3f3f3f3f3f3f3f;//LLINF
const int MAXN=(int)1e5+3;

inline char nc(){static char buf[100000],*p1=buf,*p2=buf;return p1==p2&&(p2=(p1=buf)+fread(buf,1,100000,stdin),p1==p2)?EOF:*p1++;}
inline int read(){int s=0,w=1;char ch=nc();while(!isdigit(ch)){if(ch=='-')w=-1;ch=nc();}while(isdigit(ch)){s=(s<<3)+(s<<1)+(ch^48);ch=nc();} return s*w;}
//inline void read(int &x){char ch=nc();x=0;while (!(ch>='0'&&ch<='9')) ch=nc();while (ch>='0'&&ch<='9') x=(x<<3)+(x<<1)+ch-48,ch=nc();}//根据参数个数自动选择
//void prt(int x){if(x<0){putchar('-');x=-x;}if(x>9)prt(x/10);putchar((char)(x%10+'0'));}

int h,m,a;

inline void work()
{
    h=read();m=read();a=read();
    if(a == h*m/2) printf("%lld\n",h*m);
    else
    {
        int g=__gcd(h-1,m);
        printf("%lld\n",g*(((a/g)<<1)|1));
    }
    return;
}

signed main()
{
    //ios::sync_with_stdio(false); cin.tie(nullptr); cout.tie(nullptr); //freopen(".in", "r", stdin);//freopen(".out", "w", stdout);
    signed T=1;//(signed)read();//scanf("%d",&T);//cin>>T;
    for(signed Case=1; Case<=T; Case++)
    {
        //printf("Case %d: ",Case);
        //while(cin>>n) work(n);
        work();
    }
    return 0;
}
```



## K [Scholomance Academy](https://codeforces.com/gym/103202/problem/K)

```cpp
#include<bits/stdc++.h>
#define int long long//__int128
#define mmst0(x) memset(x,0,sizeof(x))
#define mmst3f(x) memset(x,0x3f,sizeof(x))
#define pb(x) emplace_back(x)
#define mkp(x, y) make_pair(x, y)
#define fi first
#define se second
using namespace std;
typedef long long ll;
typedef long double rld;
typedef unsigned long long ull;

const rld eps = 1e-6;
const int INF=0x3f3f3f3f;//0x3f3f3f3f3f3f3f3f;//LLINF
const int MAXN=(int)1e6+3;

//int read(){int s=0,w=1;char ch=getchar();while(!isdigit(ch)){if(ch=='-')w=-1;ch=getchar();}while(isdigit(ch)){s=(s<<3)+(s<<1)+(ch^48);ch=getchar();} return s*w;}
//void prt(int x){if(x<0){putchar('-');x=-x;}if(x>9)prt(x/10);putchar((char)(x%10+'0'));}

struct Node
{
    bool c;
    int s;
}a[MAXN];

int n,tpfn,tnfp;

void work()
{
    //scanf("%d",&n);
    cin>>n;
    for(int i=1;i<=n;i++)
    {
        char c; //scanf("%c %d",c,&a[i].s);
        cin>>c>>a[i].s;
        a[i].c=(c=='+');
        //printf("111");
    }
    for(int i=1;i<=n;i++)
    {
        if(a[i].c) tpfn++;
        else tnfp++;
    }
    sort(a+1,a+1+n,[](Node a,Node b){
        if(a.s == b.s){
            if(b.c) return false;
            if(a.c) return true;
        }
        return a.s<b.s;
    });
    int tp=tpfn,fp=0,cnt=0;
    for(int i=1;i<=n;i++)
    {
        if(a[i].c) tp--;
        else cnt+=tp;
    }
    rld ans=(rld)cnt / (tpfn*tnfp);
    printf("%.9LF\n",ans);
    return;
}

signed main()
{
    //ios::sync_with_stdio(false); cin.tie(nullptr); cout.tie(nullptr); //freopen(".in", "r", stdin);//freopen(".out", "w", stdout);
    signed T=1;//(int)read();
    for(signed Case=1; Case<=T; Case++)
    {
        //printf("Case %d: ",Case);
        //while(cin>>n)
        work();
    }
    return 0;
}
```

