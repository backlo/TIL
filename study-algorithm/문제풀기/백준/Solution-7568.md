# 백준 - Solution7568

<hr>

문제링크 : [https://www.acmicpc.net/problem/7568](https://www.acmicpc.net/problem/7568)

* Java

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Solution7568 {

    private static String solution(List<Person> persons) {

        for (int i = 0; i < persons.size(); i++) {
            Person person = persons.get(i);
            for (Person anotherPerson : persons) {
                person.compare(anotherPerson);
            }
        }

        String result = String.join(" ", persons.toString());

        return result.substring(1, result.length() -1);
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        List<Person> persons = new ArrayList<>();

        int numberOfPerson = Integer.parseInt(scanner.nextLine());

        for (int i = 1; i <= numberOfPerson; i++) {
            String[] personInfos = scanner.nextLine().split(" ");
            persons.add(new Person(Integer.parseInt(personInfos[0]), Integer.parseInt(personInfos[1]), 1));
        }

        System.out.println(solution(persons));
    }

    private static class Person {
        private final int weight;
        private final int height;
        private int rank;

        Person(int weight, int height, int rank) {
            this.weight = weight;
            this.height = height;
            this.rank = rank;
        }

        void compare(Person comparePerson) {
            if (this.weight < comparePerson.weight && this.height < comparePerson.height) {
                ++this.rank;
            }
        }

        @Override
        public String toString() {
            return String.valueOf(rank);
        }
    }
}
```

