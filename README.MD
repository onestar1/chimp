#### ����
17�꿪ʼ����˾��ʼ��DotNet Coreת�ͣ����ORM���ߵ�ѡ�ͣ���ʱΧ��Dapper��EF�����˼��ҵ����ۡ���Ŀ�ŶӸ��ӹ�ע���ٽ�������������ʹ��EF�����ܿ��ٿ�����ORM����;������ҵ���ŶӶ������и��ߵ�Ҫ�����Ǹ�ϣ��ʹ����ֱ��ִ��Sql����Dapper�����ɿ��Ը��ߡ������ڼܹ��Ŷ���˵�����㿪���Ŷӵĸ�������������ǵĿ���Ч��������ĵļ�ֵ���ڣ����Ե�ʱ������һ������͵ļ�֧��EF��֧��dapper�����ݲִ���

#### Ϊʲôѡ��EF+Dapper
Ŀǰ��˵EF��Dapper��.NETƽ̨��������ORM���ߣ��Ŷӳ�Ա�Ľ��̶ܳȺܸߣ�ѧϰ�ɱ���ͣ���Ϊ��������������ص����Ϸǳ���ȫ�����ֿ�Ҳ���١�

#### ����
1. ������һ��ORM���ߣ��������κι������ݵײ�Ĳ���
2. ����һ�����׷�װ�����ݿ�ִ��͹�����Ԫģ��
3. �ܰ�������ٵĹ�����Ŀ�����ݷ��ʲ�
4. ������2���ʱ�䣬10����Ŀ�飬��С��100���������Ŀ�Ŀ���
5. ֧��EF��Dapper����������Ŀ�������л�ʹ��
6. ֧�ֹ�����Ԫģʽ��Ҳ֧������
7. ֧��Mysql��Mssql
8. ֧��ͬ�����첽�������Ƽ�ʹ���첽
> PS: �򵥲���ʹ��EF������sql����ʹ��Dapper�ǿ��ٿ������ؾ���

#### ʹ�÷���
����nuget
```

```
����ʵ����󣬼̳�IEntity

```
public class School : IEntity
{
    public Guid Id { get; set; }
    public string Name { get; set; }
}
```
�����ִ��ӿں�ʵ���࣬�ֱ�̳�IRepository��EfCoreRepository
```
public interface ISchoolRepository : IRepository<School>
{
}
public class SchoolRepository: EfCoreRepository<School>,ISchoolRepository
{
    public SchoolRepository(DbContext context) : base(context)
    {
    }
}
```
���������ģ��̳�BaseDbContext������㲻��Ҫ���������Ŀ��Բ�������һ��
```
public class KingDbContext : BaseDbContext
{
    public KingDbContext(DbContextOptions options) : base(options)
    {
    }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        //your code
    }
}
```
ע�����
```
services.AddKing<KingDbContext>(
                opt =>
                opt.UseSqlServer("Server=10.0.0.99;Database=king;Uid=sa;Pwd=Fuluerp123")
);
```
�����û�д���������
```
services.AddKing(
                opt =>
                opt.UseSqlServer("Server=10.0.0.99;Database=king;Uid=sa;Pwd=Fuluerp123")
);
```
��Controller��ʹ��
```
public class ValuesController : ControllerBase
{
    private readonly ISchoolRepository _schoolRepository;
    private readonly IUnitOfWork _unitOfWork;
    public ValuesController(ISchoolRepository schoolRepository, IUnitOfWork unitOfWork)
    {
        _schoolRepository = schoolRepository;
        _unitOfWork = unitOfWork;
    }
}
```
#### ��ϸʹ��˵��
��ѯ
```
//����������ѯ
_schoolRepository.GetById(Id)
```
```
//����׷�ٵĲ�ѯ���������ݲ������ڸ��»�ɾ�����������ܿ�
schoolRepository.TableNoTracking.First(x => x.Id == Id);
```
```
//��׷�ٵĲ�ѯ���������ݿ������ڸ��»�ɾ����������������
schoolRepository.Table.First(x => x.Id == Id);
```
```
//��ҳ��ѯ
_schoolRepository.TableNoTracking.ToPagedList(1,10);
```
```
//sql����ѯ
_unitOfWork.QueryAsync<School>("select * from school");

```
```
//sql��ҳ��ѯ
_unitOfWork.QueryPagedListAsync<School>(1, 10, "select * from school order by id");
```
> ���ڲ�ѯ����¶�˷���IQueryable��TableNoTracking��Table���������ԣ��ÿ�����Ա�Լ���װLambda����ʽ���в�ѯ����

����
```
//������֧����������
_schoolRepository.Insert(school);
_unitOfWork.SaveChanges();
```
```
//sql�������
_unitOfWork.ExecuteAsync("insert school(id,name) values(@Id,@Name)",
                        school);
```
�༭
```
//�༭��֧�������༭
var school = _schoolRepository.GetByIdAsync(Id);
school.Name="newschool";
_schoolRepository.Update(school);
_unitOfWork.SaveChanges();
```
```
//�༭�������Ȳ�ѯ
var school = new School
{
    Id = "xxxxxx",
    Name = "newschool"
};
_schoolRepository.Update(school, x => x.Name);
_unitOfWork.SaveChanges();
```
```
//sql���༭
_unitOfWork.ExecuteAsync("update school set name=@Name where id=@Id",
                        school);
```
ɾ��
```
//ɾ����֧������ɾ��
_schoolRepository.Delete(school);
_unitOfWork.SaveChanges();
```
```
//����lambdaɾ��
_schoolRepository.Delete(x => x.Id == Id);
_unitOfWork.SaveChanges();
```
����
```
//������Ԫģʽʹ������
await _schoolRepository.InsertAsync(school1);
await _schoolRepository.InsertAsync(school1);
await _unitOfWork.SaveChangesAsync();
```
```
//dapperʹ������
using (var tran = _unitOfWork.BeginTransaction())
{
    try
    {
        await _unitOfWork.ExecuteAsync("insert school(id,name) values(@Id,@Name)",
            school1,tran);
        await _unitOfWork.ExecuteAsync("insert school(id,name) values(@Id,@Name)",
            school2,tran);
        tran.Commit();
    }
    catch (Exception e)
    {
        tran.Rollback();
    }
}
```
```
//dapper+ef���ʹ������
using (var tran = _unitOfWork.BeginTransaction())
{
    try
    {
        await _schoolRepository.InsertAsync(school1);
        await _unitOfWork.SaveChangesAsync();

        await _unitOfWork.ExecuteAsync("insert school(id,name) values(@Id,@Name)",
            school2);
        tran.Commit();
    }
    catch (Exception e)
    {
        tran.Rollback();
    }
}
```
�߼��÷�
```
//ͨ��GetConnection����ʹ�ø���dapper��չ�ķ���
_unitOfWork.GetConnection().QueryAsync("select * from school");
```
#### д�����
Chimp�����ǻ���EF��Dapper�ģ�����EF��Dapperһ�й��ܶ�����ʹ�á����絼�����ԣ��ֶ�ӳ��ȵȡ��������������Ŀ�����������᳤�ڸ���ά����ϣ�������������õ������
#### ��Ŀ��ַ
[github��ַ](https://github.com/longxianghui/chimp.git)