---
title: Android 一个应用多个数据库
date: 2018-1-6 00:42:00
categories: "Android"
tags:
     - Android
---

<meta name="referrer" content="no-referrer" />





最近在做一个 IM 的项目，需要存储大量数据到本地数据库。考虑到同一台手机可能会被多个账号登录使用，为了提升数据库查询的效率，以分库的方式来存储不同账号的数据（使用用户账号来作为数据库名称）。

以存储用户信息为例：

- 先贴出使用代码：

```Java
mUserDAO = new UserDAO(this, account); // 此处的 account 就是要操作的数据库名称
mUserDAO.insert(new User(account, userName));
```

- 以下为三个关键类

```Java
/**
 * 数据库帮助类
 *
 * @author zch
 * @since 2018-01-05
 */
public class DBHelper extends SQLiteOpenHelper {

    private static final int DB_VERSION = 1;
    public static final String TABLE_NAME = "user";

    public DBHelper(Context context, String dbName) {
        super(context, dbName, null, DB_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        String sql = "create table if not exists " + TABLE_NAME + " (account text primary key , userName text)";
        db.execSQL(sql);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        String sql = "drop table if exists " + TABLE_NAME;
        db.execSQL(sql);
        onCreate(db);
    }
}
```

```Java
/**
 * 用户实体类
 *
 * @author zch
 * @since 2018-01-05
 */
public class User {

    public String account; // 用户账号，假设唯一，用它作为数据库名称（dbName）
    public String userName;

    public User(String account, String userName) {
        this.account = account;
        this.userName = userName;
    }
}
```

```Java
/**
 * 用户数据表相关操作
 *
 * @author zch
 * @since 2018-01-05
 */
public class UserDAO {

    private DBHelper mDBHelper;

    public UserDAO(Context context, String dbName) {
        mDBHelper = new DBHelper(context, dbName);
    }

    /**
     * 插入一条数据
     *
     * @param user
     * @return
     */
    public boolean insert(User user) {
        SQLiteDatabase db = null;
        try {
            db = mDBHelper.getWritableDatabase();
            db.beginTransaction();
            ContentValues values = new ContentValues();
            values.put("account", user.account);
            values.put("userName", user.userName);
            db.insertOrThrow(DBHelper.TABLE_NAME, null, values);
            db.setTransactionSuccessful();
            return true;
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            if (null != db) {
                try {
                    db.endTransaction();
                    db.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        return false;
    }

    /**
     * 删除一条数据
     *
     * @param user
     * @return
     */
    public boolean delete(User user) {
        SQLiteDatabase db = null;
        try {
            db = mDBHelper.getWritableDatabase();
            db.beginTransaction();
            db.delete(DBHelper.TABLE_NAME, "account = ?", new String[]{user.account});
            db.setTransactionSuccessful();
            return true;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (null != db) {
                try {
                    db.endTransaction();
                    db.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        return false;
    }

    /**
     * 获取所有数据
     *
     * @return
     */
    public List<User> getUserList() {
        SQLiteDatabase db = null;
        Cursor cursor = null;

        try {
            db = mDBHelper.getReadableDatabase();
            cursor = db.query(DBHelper.TABLE_NAME,
                    new String[]{"account", "userName"},
                    null,
                    null,
                    null, null, null);

            if (cursor.getCount() > 0) {
                List<User> userList = new ArrayList<>();
                while (cursor.moveToNext()) {
                    User user = new User(cursor.getString(cursor.getColumnIndex("account")), cursor.getString(cursor.getColumnIndex("userName")));
                    userList.add(user);
                }
                return userList;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (null != cursor) {
                try {
                    cursor.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            if (null != db) {
                try {
                    db.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }
}
```