#include <iostream>
#include <map>
#include <string>
#include <vector>
#include <algorithm>
#include <cstdio>
using namespace std;

// 学员账号结构体
struct Student
{
    int id;                 //账号编号
    string username;        //用户名
    int enter_year;         //入学年份
    string password_raw;    //原始密码
    string digest;          //激活校验摘要
    string status;          //账号状态 INACTIVE / ACTIVE / BAN
    int score;              //学员分数
    int register_time;      //注册时间戳
    int expire_time;        //激活过期时间戳
    bool archive;           //归档标记，true=逻辑删除
};

map<int, Student> mp;

// 根据用户名+密码计算16位摘要串
string calc_digest(const string &U, const string &P)
{
    string S = U + P;
    const long long MOD = 911382629;
    long long val = 131313;
    for (char c : S)
    {
        val = (val * 131 + c) % MOD;
    }
    char buf[20];
    sprintf(buf, "%016llx", (unsigned long long)val);
    return string(buf);
}

// 全局过期扫描：每条指令执行前调用
void expire_check(int T)
{
    for (auto &p : mp)
    {
        Student &s = p.second;
        // 未归档、已激活、到达过期时间 → 退回未激活
        if (!s.archive && s.status == "ACTIVE" && s.expire_time <= T)
        {
            s.status = "INACTIVE";
        }
    }
}

// 注册账号
void op_register(int T)
{
    int id, ey;
    string user, pwd;
    cin >> id >> user >> ey >> pwd;
    Student st;
    st.id = id;
    st.username = user;
    st.enter_year = ey;
    st.password_raw = pwd;
    st.digest = calc_digest(user, pwd);
    st.status = "INACTIVE";
    st.score = 0;
    st.register_time = T;
    st.expire_time = 0;
    st.archive = false;
    mp[id] = st;
}

// 账号激活
void op_act()
{
    int id, vl;
    string sd;
    cin >> id >> sd >> vl;
    if (!mp.count(id))
    {
        cout << "ACT_FAIL\n";
        return;
    }
    Student &s = mp[id];
    // 归档 / 状态不对 / 摘要不匹配，激活失败
    if (s.archive || s.status != "INACTIVE" || s.digest != sd)
    {
        cout << "ACT_FAIL\n";
        return;
    }
    s.status = "ACTIVE";
    s.expire_time = vl;
    cout << "ACT_SUCCESS\n";
}

// 封禁账号
void op_ban()
{
    int id;
    cin >> id;
    if (!mp.count(id)) return;
    Student &s = mp[id];
    if (s.archive) return;
    if (s.status == "ACTIVE")
    {
        s.status = "BAN";
    }
}

// 解除封禁
void op_unban()
{
    int id;
    cin >> id;
    if (!mp.count(id)) return;
    Student &s = mp[id];
    if (s.archive) return;
    if (s.status == "BAN")
    {
        s.status = "INACTIVE";
    }
}

// 修改密码
void op_mod_pwd()
{
    int id;
    string np;
    cin >> id >> np;
    if (!mp.count(id)) return;
    Student &s = mp[id];
    // 归档或者封禁账号禁止改密码
    if (s.archive || s.status == "BAN") return;
    s.password_raw = np;
    s.digest = calc_digest(s.username, np);
    s.status = "INACTIVE";
    s.expire_time = 0;
}

// 修改入学年份
void op_mod_year()
{
    int id, ny;
    cin >> id >> ny;
    if (!mp.count(id)) return;
    Student &s = mp[id];
    if (s.archive) return;
    // 只有ACTIVE允许修改
    if (s.status == "ACTIVE")
    {
        s.enter_year = ny;
    }
}

// 设置分数
void op_score()
{
    int id, sc;
    cin >> id >> sc;
    if (!mp.count(id)) return;
    Student &s = mp[id];
    if (s.archive) return;
    if (s.status == "ACTIVE")
    {
        s.score = sc;
    }
}

// 查询账号信息
void op_query()
{
    int id;
    cin >> id;
    if (!mp.count(id))
    {
        cout << "NONE\n";
        return;
    }
    Student &s = mp[id];
    if (s.archive)
    {
        cout << "NONE\n";
        return;
    }
    cout << s.id << " " << s.username << " " << s.enter_year << " "
         << s.status << " " << s.score << " " << s.register_time << " " << s.expire_time << "\n";
}

// 归档清理CLEAN操作
void op_clean(int T)
{
    int amax;
    cin >> amax;
    // 第一步：打归档标记
    for (auto &p : mp)
    {
        Student &s = p.second;
        if (s.archive) continue;
        bool c1 = (s.status == "INACTIVE") && (T - s.register_time > 2000);
        bool c2 = (s.status == "BAN") && (T - s.register_time > 1500);
        if (c1 || c2)
        {
            s.archive = true;
        }
    }
    // 收集全部归档账号，按注册时间升序
    vector<pair<int, int>> arc;
    for (auto &p : mp)
    {
        if (p.second.archive)
        {
            arc.emplace_back(p.second.register_time, p.first);
        }
    }
    sort(arc.begin(), arc.end());
    // 超量就删除注册时间最早的归档账号
    while ((int)arc.size() > amax)
    {
        int delid = arc[0].second;
        mp.erase(delid);
        arc.erase(arc.begin());
    }
}

int main()
{
    cout << "This Application is running.\n";
    int T = 0;
    string op;
    while (cin >> op)
    {
        T++;
        expire_check(T);    //每条指令前先做过期检测
        if (op == "REGISTER") op_register(T);
        else if (op == "ACT") op_act();
        else if (op == "BAN") op_ban();
        else if (op == "UNBAN") op_unban();
        else if (op == "MOD_PWD") op_mod_pwd();
        else if (op == "MOD_YEAR") op_mod_year();
        else if (op == "SCORE") op_score();
        else if (op == "QUERY") op_query();
        else if (op == "CLEAN") op_clean(T);
    }
    return 0;
}
